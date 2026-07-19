# Data Model: 001-init-from-directus (undrlla-core, Tier-A)

**Spec**: `spec.md` (same folder) | **Directus**: 12.x (v12.1.1) | **Snapshot**: `schema.snapshot.yaml`

This is the metadata-driven core data model for the Tier-A marketplace engine. Collections map 1:1 to Directus collections; relations use Directus M2O/O2M/M2M fields. Access is enforced by Directus **policy/permission row filters** and **field permissions** (FLS) — never by application code alone (FR-003, FR-004).

## Tenancy & Policy Model

- **Instance isolation (hard boundary)**: each client marketplace is its own Directus instance/container (FR-015). Cross-client isolation does NOT rely on `tenant_id`.
- **Within-instance tenancy**: `tenant_id` = a sub-organization/storefront (vendor). Vendor-owned collections carry `tenant_id` and are filtered by `tenant_id == $CURRENT_USER.tenant_id` for Vendor role.
- **Customer-scoped cross-tenant exception (FR-031)**: `Cart`/`CartItem` and a customer's own `Order` view span vendors and are filtered by **customer identity** (`customer_id == $CURRENT_USER.id`, or `anon_token` match for guests) — NOT by `tenant_id`. Vendors never read the whole cross-tenant cart; after checkout they read only `Order`/`OrderSplit` rows carrying their `tenant_id`.
- **FLS**: sensitive columns (e.g. `Product.wholesale_margin`, `Product.internal_supplier_notes`, payout secrets) are stripped for Customer/Vendor policies.
- **Indexes**: `tenant_id`, `owner_id`, `customer_id`, `status`, `checkout_group_id`, `anon_token`.

## Sensitive data handling (FR-036)

- Delivery addresses, guest `contact_handle`, Telegram chat IDs, VAT numbers/VIES responses, and payout wallet addresses are sensitive operational PII.
- API input uses structured fields (`AddressInput`, `TaxProfileInput`) rather than untyped blobs. Validation happens at the `/store/*` boundary before data reaches Directus collections.
- Field permissions mask or omit PII by role: vendors see only fulfilment data for their own `tenant_id` orders; customers see only their own address/contact/tax data; admins may see full records for support/audit.
- Audit logs and `NotificationDelivery.last_error` MUST redact raw contact handles, full addresses, VAT numbers, and payout wallet addresses. Use stable record IDs and short suffixes for support correlation.
- Retention is per instance: delivery/contact/VAT data is retained only for fulfilment, refund, tax, and statutory periods, then deleted or anonymized by scheduled maintenance.

## Roles → filter summary

| Role | Cart/CartItem | Order | Product/Service | PaymentAttempt |
| ---- | ------------- | ----- | --------------- | -------------- |
| Admin | all | all | all | all |
| Vendor | none (no cross-tenant read) | `tenant_id` match | `tenant_id` match | via own `Order` |
| Customer | own (`customer_id`/`anon_token`) | own (`customer_id`) | public read | own orders only |

## Collections

### Tenant

`id`, `name`, `domain`, `branding_config`. Sub-org/storefront within the instance (not a client boundary).

### Product

`id`, **`tenant_id`** (M2O Tenant), `title`, `description`, `type` (`physical`|`digital`), `price` (canonical currency), `inventory_quantity`, `download_asset` (digital), `wholesale_margin` (FLS), `internal_supplier_notes` (FLS), `custom_fields`. O2M → Variant.

### Variant

`id`, `product_id` (M2O Product), `tenant_id`, `sku`, `title`, `price_delta`, `inventory_quantity`, `attributes`.

### Service

`id`, `tenant_id`, `provider_id` (M2O user), `title`, `price`/`hourly_rate`, `payment_policy` (`prepaid`|`pay_later_free`), `availability` (slot rules). O2M → Booking.

### Booking

`id`, `tenant_id`, `service_id` (M2O), `provider_id`, `customer_id`, `starts_at`, `ends_at`, `status` (`pending`|`confirmed`|`rejected`|`cancelled`|`rescheduled`), `payment_attempt_id` (M2O, nullable), `rescheduled_from_id` (self M2O). Confirm atomically reserves slot and rejects overlapping `pending` (FR-019/FR-030).

### Cart  *(customer-scoped, cross-tenant — FR-031)*

`id`, `customer_id` (nullable), `anon_token` (high-entropy secret, FR-034), `anon_token_expires_at`, `contact_handle`, `contact_verified`, `updated_at`. O2M → CartItem.

### CartItem  *(cross-tenant — FR-031)*

`id`, `cart_id` (M2O Cart), `product_id`, `variant_id` (nullable), `tenant_id` (of product), `quantity`. Filtered by parent cart ownership, not `tenant_id`.

### Order  *(single-vendor — FR-024)*

`id`, **`tenant_id`**, `checkout_group_id`, `customer_id` (nullable for guest), `customer_contact`, `total_amount`, `platform_fee_amount`, `tax_amount`, `tax_mode` (`inclusive`|`exclusive`), `reverse_charge`, `status`, `payment_method`, `fulfilment_status`, `tracking_number`, `delivery_method` (`delivery`|`pickup`), structured `delivery_address` (`country`, `region`, `city`, `line1`, `line2`, `postal_code`), `shipping_amount`, `delivery_zone_id` (M2O). O2M → OrderItem. A checkout group may hold mixed statuses; each order remains independently paid/failed/refunded.

### OrderItem

`id`, `order_id`, `tenant_id`, `product_id`, `variant_id`, `quantity`, `unit_price`, `line_tax_amount`.

### OrderSplit  *(fan-out — FR-024/FR-031)*

`id`, `checkout_group_id`, `tenant_id`, `order_id` (M2O), `payment_attempt_id` (M2O), `platform_fee_amount`, `status` (`pending`|`paid`|`failed`|`refunded`). One row per vendor in a multi-vendor checkout; vendor-filtered by `tenant_id`. The customer-facing checkout group derives aggregate mixed states (`partially_paid`, `partially_failed`, `partially_refunded`) from these rows.

### PaymentAttempt  *(provider-agnostic — FR-013/FR-032)*

`id`, `order_id`, `provider` (`stripe`|`ton`|`shkeeper`), `status`, `settlement_currency`, `quoted_rate`, `crypto_amount`, `received_amount`, `amount_mismatch_state` (`ok`|`underpaid`|`overpaid`|`expired`), `quote_expires_at`, `idempotency_key`, `provider_payment_id`, `provider_event_id`, `refund_amount`, `settlement_model` (`stripe_connect_application_fee`|`crypto_collect_and_forward`), `payout_wallet_snapshot`. For TON/SHKeeper v1, `payout_wallet_snapshot` freezes the vendor destination at attempt creation; later wallet changes affect only future attempts.

### RefundRequest  *(FR-020)*

`id`, `order_id`, `payment_attempt_id`, `requested_amount`, `approved_amount`, `status` (`requested`|`approved`|`rejected`|`provider_confirmed`|`provider_failed`), `reason`, `approved_by`. `approved_amount ≤ captured`; only `provider_confirmed` mutates payment/order state.

### InventoryReservation  *(FR-018)*

`id`, `order_id`, `product_id`, `variant_id`, `quantity`, `status` (`held`|`consumed`|`released`), `expires_at`. Atomic hold at checkout; consumed on confirmed payment; released on cancel/fail/TTL.

### DeliveryZone  *(FR-021)*

`id`, `tenant_id`, `name`, `address_match_rules`, `rate_type` (`fixed`|`free`), `rate_amount`, `free_from_amount`. No match → delivery unavailable (no fallback rate).

### DigitalEntitlement  *(FR-018)*

`id`, `order_id`, `product_id`, `customer_id`, `status` (`active`|`revoked`|`expired`), `download_token`, `expires_at`, `revoked_at`. Created only after confirmed payment; protected time-limited link; admin/vendor revocable.

### VendorPayoutAccount  *(FR-025)*

`id`, `tenant_id`, `provider`, `status`, `stripe_connect_account_id`, `payout_wallet_address`, `onboarding_completed_at`. Provider offered at checkout only when complete. Secrets injected per FR-035 (not stored here).

### TaxRegistration  *(FR-029)*

`id`, `jurisdiction`, `tax_provider`, `collect_enabled`, `nexus_type`, `threshold_amount`.

### TaxProfile / TaxQuote  *(FR-029/FR-036)*

Transient checkout input/output, not a broad customer profile by default. `TaxProfile` carries `jurisdiction`, `buyer_type`, and optional VAT number for VIES validation. Store only the minimum tax evidence required for statutory compliance on the `Order`; redact raw VAT values from logs and non-admin reads.

### TelegramLink / NotificationDelivery  *(FR-026/FR-033)*

- TelegramLink: `id`, `user_id`, `telegram_chat_id`, `linked_at`.
- NotificationDelivery: `id`, `event_type`, `recipient_user_id`, `recipient_role`, `channel` (`telegram`|`email`), `status`, `attempt_count`, `last_error`, `next_retry_at`, `sent_at`.

### BrandingSettings

`tenant_id`, `primary_color`, `secondary_color`, `theme_name`, `logo_url` (from `@underundre/undesign`).

## Tier-B (deferred to `002+`, FR-016 scaffolding only)

MutualAidQueue, VpnNode, BarterItem, TaxiRequest are NOT created in `001`. Only the extension-package seams that would host them (U-T13) ship in this feature; a clean Tier-A instance contains none of these collections by default.

## Key relations

- Tenant has many Product, Service, Order, DeliveryZone, and VendorPayoutAccount records.
- Cart has many CartItem records; Order has many OrderItem records and one PaymentAttempt.
- checkout_group_id groups N Orders / OrderSplits / PaymentAttempts (one per vendor)
- Service has many Booking records; Booking has zero or one PaymentAttempt.
- Order has many RefundRequest, DigitalEntitlement, and InventoryReservation records.
