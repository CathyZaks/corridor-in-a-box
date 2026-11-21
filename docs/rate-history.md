# Rate history

Where do the prices the corridor engine uses come from, and how fresh are
they? This page documents the corridor-in-a-box rate surface: the
sources, the cadence, and the licences. (Written for
[corridor-in-a-box#601](https://github.com/corridor-in-a-box/corridor-in-a-box/issues/601);
adapted to the corridor codebase — see "Source-name map" below.)

## Sources

### SEP-38 quote server (the corridor's quote source)

- **Measures:** the exchange rate of the corridor's `fx.path` (e.g.
  `["USD", "USDC", "USD"]` on `corridors/reference.corridor.yaml`;
  `["USD", "ZAR"]` on `corridors/zeam.corridor.yaml`).
- **How fetched:** on demand, by the engine's `quote` verb, via SEP-38
  `POST /quote` against the destination anchor's quote server — the URL
  is the corridor manifest's `dest.endpoints.quote_server`
  (`http://localhost:8080/sep38` on the reference lane,
  `https://anchor.zeam.money/sep38` on Zeam). Corridor does not collect
  or store rates: quotes are ephemeral per corridor run.
- **Firm vs estimated:** `fx.quote_source: sep38` on the firm lanes
  (`zeam`, `reference-testnet`) requests firm quotes (`sep38_firm`) —
  they carry a future expiry, expired by the engine at
  `fx.quote_ttl_seconds` (60 s on the reference lane). Lanes with
  `quote_source: external` (e.g. `ng-cowrie`) have no SEP-38 source at
  all. Indicative quotes (`sep38_indicative`) are estimates by
  definition; corridor's engine surfaces quote failures rather than
  inventing numbers.
- **Anchor domain and network:** read from the corridor manifest —
  `dest.endpoints.home_domain` etc. (`anchor.zeam.money`, the reference
  server's `localhost:8080`). Corridor trusts the manifest as configured;
  it does not re-resolve or cache the domain.

### Source-name map to corridor reality

The issue this page was written for lists rate sources — `sep38_price`,
`dex_trade`, `reference_fx`, `own_payment` — that corridor does not
collect (there are no `dex_trade`, `reference_fx` or `own_payment` or
`sep38_price` values anywhere in the repo). Corridor's rate source is the
anchor quote server above — the closest named corridor source is the
SEP-38 quote (the manifest `fx.quote_source`, the `sep38_firm` /
`sep38_indicative` channel quote types in `packages/adapter-kit` and
[docs/adapters.md](./adapters.md)). Any future rate collection that does
have `dex_trade` / `reference_fx` / `own_payment`-style sources should
re-open this page.

## Collection interval

There is no collection cadence — quotes are requested during each
corridor run (the `quote` verb), subject to the manifest's
`fx.quote_ttl_seconds`. `packages/engine` has no scheduler and
`packages/service` runs the engine per request; nothing polls or stores
rates on a timer.

## Retention

Corridor stores no rate samples and computes no daily summaries, so
there is no retention policy to document. The records that do persist
are per corridor: the manifest `id`, `endpoints_verified_at` (when the
anchor check was recorded, e.g. 2026-10-17 on Zeam) and the `proof:`
block written by `corridor canary` after a proven settlement.

## Minimum-data rule

Corridor has no "estimated price" path — quotes are either firm or
absent, and `corridor plan` surfaces the absence (for example:
"fx.quote_source=sep38 but dest exposes no SEP-38 quote server — quotes
will fail"). When the corridor requires quotes and the adapter reports
`quotes: ["none"]`, execution fails with `CAPABILITY_MISMATCH`
(see CHANGELOG).

## Daily summary

Corridor computes no daily rate summaries. The per-run verb sequence is
`quote → comply → settle → reconcile → recover` (see
[README.md](../README.md)); rates exist only as the quotes fetched at the
start of a run.

## Licence per source

- **SEP-38 quotes (anchor quote servers):** the Stellar SEP-38 protocol
  is public; quotes come from each anchor's own quote server. Before
  republishing or redistributing any quote data, check the anchor's
  terms and its `stellar.toml` — those are the governing terms; corridor
  itself collects and stores no rate data, so there is no corridor rate
  licence.
- **Anchor SEP-38 quotes:** each anchor's terms.
- **Reference FX / third-party FX providers (if added later):** check the
  provider's redistribution licence for limits — anything unconfirmed is
  marked "to confirm", never assumed.
- **DEX / other data sources:** not present in corridor today.

## To confirm

- Whether anchors permit quote redistribution — check each anchor's
  `stellar.toml` and the SEP-38 quote API before reusing any quote
  values.
- The licence terms for any future external rate sources named
  `dex_trade`, `reference_fx`, `own_payment` by the original issue.

## Links

- [corridors/](../corridors/) — corridor manifests (`fx`,
  `dest.endpoints`)
- [docs/adapters.md](./adapters.md) — adapter quote types
  (`sep38_firm`, `sep38_indicative`)
- [README.md](../README.md) — the corridor sequencing table (which
  corridors use SEP-38 quotes and which use `quote_source: external`)
