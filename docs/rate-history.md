# Rate history

Where the corridor engine records and summarizes the prices it uses. This
page documents the rate history that RFC 0004 §3
([docs/rfcs/0004-smart-router.md](rfcs/0004-smart-router.md#3-rate-history))
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
  - Fetched from: the provider of the reference FX rates; the provider's
    licence must allow republishing, and corridor stores no source-licence
    data.
- **Own payment** — the business's own completed payments.
  - Measures: the effective rate (`amount_out / amount_in`) of the
    business's own completed payments — the only data for anchors that
    publish no price. It is not the bridge asset's rate from the manifest.
  - Fetched from: corridor records the completed payments; status: to confirm —
    no live fetch is implemented yet.

## Collection

Every 15 minutes (RFC 0004 §3): samples are collected into Postgres (the
engine's store); the destination table and schema: to confirm.

## Daily summary

Per RFC 0004 §3, a daily summary per pair is the **min, P25, median, P75,
max and sample count**, published as JSON so anyone can check the numbers
the router uses. The collector that gathers the samples is an always-on,
read-only service, not GitHub Actions.

## Retention and minimum-data rule

An anchor gets estimated only with at least 7 days of data and at least 100
samples in the window; below that it is unpriceable. Retention is not decided,
so mark it "to confirm".

## Links

- [README.md](../README.md) — the docs index
