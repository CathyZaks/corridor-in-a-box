# Rate history

Where the corridor engine records and summarizes the prices it uses. This
page documents the rate history that RFC 0004 §3
([docs/rfcs/0004-smart-router.md](docs/rfcs/0004-smart-router.md))
specifies: four rate sources, a 15-minute collection cadence into
Postgres, a daily summary, and a retention/minimum-data rule.

## Sources

Per RFC 0004 §3, each source is recorded separately (never mixed):

- **SEP-38 indicative** — anchor-published SEP-38 quotes.
  - Measures: the exchange rate the corridor's `fx.path`
    (e.g. `["USD", "USDC", "USD"]`) pays, as an _indicative_ (estimated)
    rate — corridor does not treat indicative quotes as firm.
  - Fetched from: the corridor's quote server
    (`dest.endpoints.quote_server` in the corridor manifest) via the
    SEP-38 `POST /quote` endpoint; anchor domain and network are recorded
    from the quote response / the anchor's published `stellar.toml` —
    status: to confirm (corridor records the domain/network per sample —
    the field to persist them in is not yet defined).
  - Licence: each anchor's terms — check the anchor's `stellar.toml` and
    quote API before republishing quote values; the Stellar SEP-38
    protocol is public and corridor stores no source-licence data.
- **DEX** — DEX trade rates.
  - Measures: live DEX-traded rates for the corridor's `fx.path`, when
    the corridor runs through a DEX path.
  - Fetched from: to confirm — the DEX data source and queries for this
    corridor are not yet defined in the codebase.
- **Reference FX** — rates from an FX reference provider.
  - Measures: FX quotes for the corridor's `fx.path` where the corridor
    pays an off-chain payout asset (e.g. `iso4217:USD`) through the bridge
    leg.
  - Fetched from: to confirm — the provider and its redistribution
    licence for corridor use are not yet decided (see Licences below).
- **Own payment** — the bridge asset's own rate
  (`settlement.bridge_asset` in the corridor manifest, e.g. USDC on the
  reference lane).
  - Measures: the bridge asset's value against the corridor's `fx.path`
    (used by the ramp leg).
  - Fetched from: the corridor's config (manifest `settlement` / `fx`
    fields) — status: to confirm — no live fetch is implemented yet.

## Collection

Every 15 minutes (RFC 0004 §3): samples are collected into Postgres (the
engine's store); the destination table and schema: to confirm.

## Daily summary

Per RFC 0004 §3, a daily summary per source is the **min, P25, median,
P75 and max** of the day's samples.

## Retention and minimum-data rule

Per RFC 0004 §3: data is kept **at least 7 days** and a source-day
requires **at least 100 samples** — details: to confirm.

## Links

- RFC 0004 §3 — the rate-history spec (RFC 0004 is not yet present in the repo docs — tracked in corridor issue #601)
- [README.md](../README.md) — the docs index
