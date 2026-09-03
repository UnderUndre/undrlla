# Quickstart: 002-provisioning-portal

## Local Setup

1. Start Directus dev instance.
2. Submit a valid manifest:

```bash
curl -X POST http://localhost:8055/provisioning/manifests \
  -H "Authorization: Bearer <operator-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "manifest_version": "1.0",
    "tenant_id": "acme-store-001",
    "display_name": "Acme Store",
    "domain": "acme.undrlla.example",
    "region": "eu-central-1",
    "billing_reference": "stripe:acct_123",
    "initial_admin_email": "admin@acme.example",
    "addons": ["payments"],
    "secrets": {
      "directus_service_token_ref": "secret:directus/service_token_env"
    }
  }'
```

3. Assert response is `202 Accepted` with `manifest_id` and `status: queued`.
