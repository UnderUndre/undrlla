# Undrlla Constitution

**Status:** v0 (draft, product law)  
**Adopted (intent):** 2026-07-27
**Not:** AI persona instructions, state law, immigration advice, or a guarantee of any national passport.

This document is the **political-product constitution** of Undrlla as a digital polity that builds tools for mobility, housing, work, and digital safety. Feature specs (`001`, `002`, `003`, …) implement pipes. This file decides **who we are** and **what we refuse**.

Cross-links: [`ECOSYSTEM.md`](./ECOSYSTEM.md) · [`ROADMAP.md`](./ROADMAP.md) · [`003-housing-donations/spec.md`](./003-housing-donations/spec.md)

---

## Preamble

Undrlla is a digital state of builders, not a prayer circle and not a shadow immigration office.

Life is high-pressure plumbing. When a pipe bursts, we find the leak and fit a clamp — we do not invent a new religion or a new nation-state on paper. Undrlla operates **tools and markets** so people can protect time, resources, and options under real laws of real countries.

We do not confuse:

1. **Membership in Undrlla** (platform status), with  
2. **Legal status in a country** (visa, residency, citizenship), with  
3. **A private relationship** (including marriage under state family law).

Mixing those three into one word is a clog. This constitution keeps the valves separate.

---

## I. Value Hierarchy (L1–L4)

Hard priority order. Lower levels do not override higher ones.

| Level | Name | Meaning for Undrlla |
| ------- | ------ | --------------------- |
| **L1** | Life & health | No product path that knowingly routes people into physical danger for growth metrics. Housing payouts go to **escrow / notary / verified vendor**, never “cash for need” as a dark pattern. |
| **L2** | Time & liberty | Prefer one login, one handle, fewer theatrical interviews, less bureaucracy theater. Tools that free time beat tools that create busywork. |
| **L3** | Resources & infrastructure | Marketplace, housing pool, undevops, unet — real economic pipes. Fees are transparent. |
| **L4** | Rules, laws & ToS | Country law, payment-network rules, platform ToS. They are clamps: ignore them and the system floods — but they are **not** higher than L1–L3. If a rule blocks survival or liberty without necessity, we redesign the pipe, not worship the clamp. |

**Operational rule:** when product, growth, and safety conflict — L1 first, then L2, then L3, then L4.

---

## II. Five Contours

Borrowed framing; Undrlla’s main work sits on contours 2–3.

1. **Personal** — body, mind, local trust circle, personal digital identity.  
2. **Business** — shops, services, workforce, undevops-hosted instances, undrecruiting.  
3. **State-interaction** — housing purchase logistics, residency/CBI **orientation**, document checklists, concierge under real jurisdictions.  
4. **Planet** — geography, logistics, climate, multi-country ServiceArea reality.  
5. **Meta** — long-horizon norms (this constitution, amendment culture, anti-scam posture).

Undrlla is **not** a sovereign state. It does not issue national passports. It **does** issue platform status, handles, and economic access under its own ToS.

---

## III. Persons (glossary)

Use these labels in product copy, specs, and UI. Prefer English enum keys in systems; Russian UI must qualify «гражданин».

| Term | System key | Means | Does **not** mean |
| ------ | ------------ | -------- | ------------------- |
| **Migrant** | `undrlla.migrant` | Every registered natural person by default: may shop, pay, apply, work, match | “Illegal immigrant”; moral inferiority |
| **Citizen** | `undrlla.citizen` | Platform member who earned citizenship under §IV | Any country’s national |
| **Applicant** | `housing_applicant` / similar | Person in a housing / papers-oriented program flow | Automatically a citizen |
| **Contributor** | (billing role) | Pays into housing pool or other ecosystem billing | Citizen by default |
| **Vendor / worker** | marketplace roles | Offers services or labor | Citizen gate |
| **Donor / subscriber** | pool payer | Funds shared housing pot (see `003`) | Control over recipient in MVP |
| **State resident / national** | off-platform legal fact | Status under a country’s law | Undrlla rank |
| **Handle** | `@username` | Global unique public name | Phone number as root of trust |

**UI rule:** never bare «гражданин» / “citizen” without scope. Always **«гражданин undrlla»** vs **«гражданство [страны]»** / “national of …”.

---

## IV. Citizenship (`undrlla.citizen`)

### IV.1 Earn rules (locked v0)

Hybrid path **A1** — clock + verification + standing:

1. **Active ecosystem billing** for **12 consecutive months** — at least one qualifying stream, for example:  
   - housing-pool subscription ≥ USD 1 / month, or  
   - unet paid plan, or  
   - undevops / platform seller or subscription fee, or  
   - other flagship recurring products designated by ops.  
2. **Verified human** — KYC-lite / liveness (provider TBD in implementation specs).  
3. **Clean standing** — no active ban; no unresolved serious ToS / fraud strike during the window.

**Auto-promote** when all three hold.  
**Lapse:** if qualifying billing is inactive **90 days**, status becomes `lapsed` (benefits pause). Re-qualify without full 12 months if lapse ≤ 90 days and KYC still valid; after 90 days, clock rules are defined in implementation (default: restart or partial credit — product may tighten later without changing the 12-month *standard*).

**Not in v0:** pay-to-skip bond/stake; contribution-score acceleration (post-MVP candidate only).

### IV.2 Benefits (v0 must-have)

Citizens receive at least:

1. **Fee discounts** on designated ecosystem products (unet, undevops, platform fees — rates in pricing docs, not here).  
2. **Voice** — participation in roadmap polls and constitution-amendment consultation (§XII).  
3. **Public trust surface** — citizen badge on profile and eligibility for elevated display on `undrlla.com/@handle`.

### IV.3 Explicit non-benefits (v0)

- **No housing-queue boost.** Shared pot FIFO / model B in `003` is not overridden by citizenship.  
- **No exclusive right to work or hire** on undrecruiting — migrants may hire and be hired. Badge is a **signal**, not a permission bit.  
- **No state legal privilege.** Citizenship does not create visa, marriage, or passport rights in any country.

### IV.4 Duties

- Obey ToS and applicable law in jurisdictions you touch.  
- No sham marriage coaching, no document fraud, no pool-abuse.  
- Do not present undrlla status as a national passport.

---

## V. Identity & Handles

### V.1 Single identity

Flagship Undrlla (Directus users) is the **IdP** for the ecosystem MVP (`ECOSYSTEM.md`). One brain for “who is this person”.

### V.2 Global handle

- One **globally unique** `@username` across undrlla, undrllanding, undrepost, unet, undevops-facing identity, and future modules.  
- Canonical public page: **`undrlla.com/@username`** (hub) — entry to modules the person uses (shop, housing, unet, posts, recruiting, matching), subject to feature flags.  
- Phone / SMS is **not** the root of trust (digital hydroseal principle). Prefer TOTP / WebAuthn / hardware keys where feasible.

### V.3 Reservation & reclaim

- System reserves: product names (`undrlla`, `unet`, …), abusive strings, and a seed list of ISO country codes / obvious geo brands.  
- **Lawful rights holders** (trademark owners, governments, verified orgs) may **claim** a handle via verified process; squatters may be forced to rename with a free rename affordance.  
- Dispute process is operational (ticket + evidence), not “who paid more”.

### V.4 Squatting & premium

Premium sales and auction of handles are **optional later**. They must not block trademark reclaim.

---

## VI. Economy

### VI.1 Marketplace

Tier-A commerce (products, services, bookings, payouts) is the economic spine. Flagship extensions plug in without poisoning clean client shops (`001` extension points).

### VI.2 Housing pool (principles)

As locked in `003` (summary; spec wins on conflict of detail):

- Shared pot + queue; donors do not earmark recipients in MVP.  
- Payout only via escrow / notary / verified vendor — **keys, not cash to applicant**.  
- Platform donation fee transparent (default 5% + payment pass-through).  
- No self-serve refund theater after successful pay (legal exceptions admin-only).

### VI.3 “No custom boiler”

Prefer proven vendors (IdP stack, Stripe/crypto rails, KYC provider, hosting) over inventing a private central bank or amateur crypto state in MVP.

---

## VII. Mobility & Papers

### VII.1 Package story

Primary flagship story remains **housing + orientation toward legal mobility outcomes** (residency / CBI / naturalization tracks where they exist). Marketing **must not** promise a passport.

### VII.2 Housing ⟂ papers country (locked)

A person may:

- target **housing purchase** in country **X**, and  
- pursue a **residency or investment-citizenship track** in country **Y**,

as they choose and as real law allows.

Product and data models **must not force** `housing_country == papers_country`. Feasibility is advisory (warn-only metadata), not a hard “system approved your passport”.

### VII.3 MVP geography

- **Live (ops-enabled):** Turkey (TR), Georgia (GE) for donation-funded housing programs unless admin disables ServiceArea.  
- **Research backlog:** Armenia, Argentina, Brazil, Egypt, UAE (residency ≠ citizenship), and others — enabled only as data + legal review, not as empty marketing promises.

### VII.4 Prohibited claims

- “Guaranteed citizenship / passport”.  
- “Undrlla is a country that naturalizes you”.  
- Selling immigration outcomes without licensed partners where required.

---

## VIII. Union (marriage-minded matching)

### VIII.1 Purpose

Optional matching for people seeking **real**, **non-sham** relationships, including those open to legal marriage under the laws of a country they choose. Logistics (meetup, concierge, housing checklists) may be offered as ordinary services.

### VIII.2 Framing

Closer to **serious dating + logistics** than to “citizenship retail”.  
Possible later brand names are product decisions; constitution only sets red lines.

### VIII.3 Red lines

- No guarantee that marriage yields residency or citizenship.  
- No product flows that coach **fictitious** cohabitation or document fraud.  
- Platform is not a substitute for licensed immigration counsel.  
- Citizen **veto** over other people’s matches is **not** a v0 right (both parties opt in; report/block only). Revisit only by amendment.

### VIII.4 Safety

Harassment, trafficking patterns, and fraud → enforcement under ToS (L1/L2). Feature design must not optimize pure volume over safety.

---

## IX. Work (undrecruiting)

### IX.1 Intent

Reduce hiring theater. Prefer **proof** (public work, OSS, licenses, completed platform jobs, short paid trials) over five rounds of HR fog.

### IX.2 MVP wedge

One talent/vendor profile schema; at least two demand sources:

1. **IT / builders** — hire fast with proof + trial.  
2. **Housing / CBI operational crew** — lawyers, realtors, translators, escorts (aligned with post-MVP multi-vendor direction in housing specs).

Optional later: OSS funding / revenue-share team formation — not required for v0 constitution.

### IX.3 Access

Migrants and citizens may both offer and buy work. Citizenship badge is reputation paint, not a union card.

### IX.4 Implementation preference

Reuse marketplace `Service` / `Booking` / payouts where possible (crew packs, staff-aug SKUs) instead of a disconnected “second LinkedIn”.

---

## X. Governance & Balance of Power (Option A: Benevolent Technocracy / Corporate Monarchy)

### X.1 Governance Model

Undrlla operates on a **Benevolent Technocracy / Corporate Monarchy (Option A)** governance model during early growth, with an explicit path toward a **Constitutional Republic / Hybrid Parliament (Option B)** as the citizen body matures. 

The founder acts as the **Hokage / Chief Architect (Steward)**. Executive decision-making is streamlined to prevent political paralysis and protect protocol stability.

---

### X.2 Matrix of Rights & Duties (Матрица прав и обязанностей)

| Entity | Rights (Права) | Duties & Guarantees (Обязанности и гарантии) |
|---|---|---|
| **Digital State (`Undrlla`)** | 1. Charge platform protocol fees (up to 15% max cap).<br>2. Trigger emergency account freeze / hold valves (`Emergency Valve`) on L1 safety incidents (fraud, exploits, spam, legal takedowns). | 1. Guarantee 99.9% target SLA on core infrastructure.<br>2. Zero-Knowledge data privacy (never sell or leak user PII to 3rd parties).<br>3. Guaranteed automated escrow payouts (85%+ net) to verified providers upon milestone completion. |
| **Hokage / Steward (Founder)** | 1. Sole executive veto on protocol changes during Option A phase.<br>2. Appointment of Lead Architects, Code Reviewers, and Arbitrators.<br>3. Emergency intervention in security/fraud incidents. | 1. Maintain technical infrastructure and zero-broken-windows code quality.<br>2. Uphold L1–L4 Value Hierarchy without exception.<br>3. Publish post-mortem audit logs for any Emergency Valve intervention within 72 hours. |
| **Migrants (`undrlla.migrant`)** | 1. Universal access to browse, shop, pay, apply for housing, and work.<br>2. Zero-Knowledge privacy protection for identity and transactions. | 1. Strict adherence to ToS and local laws.<br>2. Prohibition of fraud, sham marriage arrangements, or pool exploitation. |
| **Citizens (`undrlla.citizen`)** | 1. Ecosystem fee discounts on unet, undevops, and marketplace SKUs.<br>2. Advisory voice in roadmap polls & constitutional consultations.<br>3. Public verified badge and elevated display on `@handle` hub.<br>4. Priority queue in dispute arbitration. | 1. Continuous active qualifying billing ($19/mo or equivalent qualifying stream).<br>2. Maintenance of clean standing (no active ToS strikes).<br>3. Optional Jury Duty in community dispute arbitration (post-MVP). |

---

### X.3 Emergency Valve Protocol (Аварийный клапан сброса давления)

For L1 incidents (security breaches, active fraud, malicious smart contract exploits, legal takedowns), the Hokage / Steward MAY immediately freeze affected accounts or pause pool allocations. 
- The emergency action takes effect instantly.
- A public post-mortem report detailing the incident and technical rationale MUST be published to the community within **72 hours**.

---

### X.4 Evolution Path to Option B (Hybrid Parliament)

When the active citizen count reaches 10,000 verified citizens, the governance model MUST initiate a transition review to **Option B (Constitutional Republic)**, introducing binding citizen polls (with 60% quorum requirements) for fee structure adjustments and constitutional amendments.

---

## XI. Security (digital hydroseal)

1. Prefer phishing-resistant auth over SMS-as-recovery root.  
2. Backups and critical data follow immutable / tested restore culture (3-2-1-1-0 spirit).  
3. Least privilege for staff and AI agents accessing user data.  
4. No dark pattern that forces unsafe identity roots for convenience alone.

---

## XII. Amendment

1. **Propose** — steward or citizen proposal published (path TBD: repo issue / governance channel).  
2. **Consult** — citizen poll when the change touches §§III–IX rights or red lines.  
3. **Adopt** — version bump (`v0` → `v0.1` / `v1`), changelog entry in this file or adjacent `CONSTITUTION-CHANGELOG.md`.  
4. **Conflict** — on pipe details, **active feature specs win** until this constitution is amended; on **values and red lines**, this constitution wins and specs must be fixed.

Editorial typos may be fixed without poll. Changing earn rules, benefits, or red lines requires a version note and consultation.

---

## XIII. Relationship to feature specs

| Topic | Authority |
| ------- | ----------- |
| Value hierarchy, dual labels, red lines | **This constitution** |
| Housing pool math, escrow, TR/GE ServiceArea | `003` (+ future amends for housing ⟂ papers fields) |
| IdP, app order, undrllanding flags | `ECOSYSTEM.md` |
| Marketplace primitives | `001` |
| Provisioning | `002` |
| Pricing numbers | pricing / ops docs (not constitutional) |

---

## Changelog

| Version | Date | Notes |
|---------|------|--------|
| **v0.1** | 2026-07-30 | Added Section X Governance & Balance of Power: Option A (Benevolent Technocracy / Corporate Monarchy) with explicit Matrix of Rights & Duties for State, Hokage/Steward, Migrants, and Citizens; 72h post-mortem requirement for Emergency Valve; 10k citizen trigger for Option B transition review. |
| **v0** | 2026-07-27 | Initial draft from brainstorm log: dual labels; citizen = 12 mo hybrid A1; benefits fees/voice/badge; no queue boost; global handles + TM reclaim; housing ⟂ papers; marriage 5A red lines; undrecruiting D wedge; TR+GE + research backlog. |

---

## Open implementation hooks (not blockers for v0 text)

- Exact fee discount table for citizens.  
- KYC-lite vendor selection.  
- Handle dispute SLA and reserved-name seed list.  
- `003` fields: `housing_country` vs `papers_track_country` — **specified 2026-07-27** (implement when coding `003`).  
- Marriage product slug / brand — see `006-union-matching.md`.  
- Binding vs advisory poll thresholds for §X.  
- Book license review if longer quotations from *Сантехника бытия* are ever embedded.  
- Marketplace template split → `undreseller` (Medusa); kill-or-keep Directus Tier-A dual path.
