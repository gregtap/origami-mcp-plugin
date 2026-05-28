---
name: france-companies
description: Use when researching French companies or their officers, such as looking up a SIREN/SIRET, reading a company fiche, pulling annual accounts, BODACC announcements, marques, AMF filings, mapping ownership networks, screening companies by finance/LBO criteria, or running a PEP check. Drives the Origami MCP tools (search_companies_tool, get_company_tool, get_dirigeant_tool, and so on) over the French SIRENE/RNE registry.
---

# Researching French companies with Origami

The Origami MCP exposes 16 tools over roughly 29M French legal units (SIRENE),
the RNE business register, annual accounts, BODACC announcements, trademarks
(marques), AMF filings, and the HATVP PEP feed. This skill is about driving
those tools well. The tool schemas already describe their arguments.

## Always resolve a SIREN first, never guess it

A SIREN is a 9-digit identifier. Do not invent or assume one. Resolve a name to
a SIREN before any detail call:

- `suggestions_tool` is fast autocomplete on company name, SIREN, or dirigeant
  name. It is the best first hop when the user typed a name.
- `search_companies_tool` adds BM25 plus filters (`q`, `naf`, `dept`, `etat`,
  `sort`) when you need to rank or filter rather than just autocomplete.

Then pass the resolved SIREN to the detail tools.

## Tool-chaining map

**Company deep-dive.** Start with `get_company_tool` for the bundled fiche
(identity, siège, establishments, dirigeants, 3-year finance summary, recent
BODACC, marques, CAC40 flag). Follow with `get_comptes_annuels_kpi_tool` for
multi-year KPIs and `search_bodacc_tool` for the full announcement history.

**People and officers.** Use `search_dirigeants_tool` (current RNE snapshot plus
BODACC historical matches), then `get_dirigeant_tool` (identity plus full
mandate list), then `get_cartographie_tool` for the 2-hop person and company
ownership or officer network.

**Screening at scale.** Use `search_companies_by_finance_quality_tool`
(BFR-trajectory pre-filter), `search_companies_by_lbo_eligibility_tool` (LBO
shape: turnover, age, profitability, leverage, collective proceedings), and
`top_rentable_companies_tool` (highest-margin SIRENs by NAF or department).

**Filings and compliance.** Use `get_amf_documents_tool` for the AMF
regulated-filings archive, `pep_check_tool` for a politically-exposed-person
check, and `get_actes_for_siren_tool` for a navigable table of contents over
INPI acte PDFs.

`get_account_tool` returns the calling key's owner, tier, and rate limit.

## Use the batch KPI tool for many companies

When you need annual-account KPIs for several SIRENs, call
`get_comptes_annuels_kpi_batch_tool` (up to 50 SIRENs per call) instead of
looping single calls. It is both faster and easier on the rate limit.

## pep_check: pass siren_context to disambiguate homonyms

`pep_check_tool` matches a person against the HATVP feed. French names produce
homonyms, so pass `siren_context` (the SIRENs the person is associated with in
the current task) to get a `match_confidence`:

- `high` means the HATVP department overlaps a company's siège department, so it
  is likely the right person.
- `low` means a department is present but does not overlap. Treat it as a
  possible homonym, phrase it as "possible homonym, to be confirmed", and do not
  assert PEP status.
- `medium` is a national-scope role (minister, deputy, senator, MEP) where
  geography is not evidence either way.
- `null` means no `siren_context` was passed.

Confidence is advisory. Narrate the match and let the human reader decide.

## Data caveats, state these honestly to the user

- **Dirigeant date of birth is `YYYY-MM` only** (privacy minimisation). Do not
  present a full birth date.
- **Dirigeant coverage leans toward current mandates.** The officer tables do
  not capture the full historical graph. If a company plainly has more officers
  than the tools return, say coverage is partial. Never claim "this company has
  only N dirigeants" as fact.
- **Bénéficiaires effectifs (RBE, beneficial owners) are access-gated** under
  INPI's "intérêt légitime" rules. A fiche may carry a notice that RBE data was
  withheld for the calling key. Surface that notice rather than implying there
  are no beneficial owners.

## Rate limits

The MCP enforces a per-key, per-minute rate limit. On a `[rate_limited]` error,
back off briefly and retry. Do not hammer. `get_account_tool` shows the current
limit.
