<div align="center">

<img src="./assets/logo_origami.svg" alt="Origami" width="120" height="120">

# Origami for Claude Code

**French company intelligence, in your terminal.**

Search and analyze 29M+ French companies (registry, officers, accounts, filings,
trademarks, compliance) straight from a Claude Code prompt.

[![Plugin](https://img.shields.io/badge/Claude%20Code-plugin-d8543a)](https://code.claude.com/docs/en/plugins)
[![MCP](https://img.shields.io/badge/MCP-streamable--http-3c825a)](https://modelcontextprotocol.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-6eb482)](./LICENSE)

</div>

---

This plugin connects Claude to the [Origami](https://origami-entreprises.fr) MCP
server. It exposes 16 tools over the French SIRENE legal-unit base and the RNE
business register: company fiches, dirigeants (officers), annual accounts,
BODACC announcements, trademarks (marques), AMF filings, INPI acte PDFs, and
HATVP PEP checks.

Ask in plain language and Claude picks the right tools and chains them for you:

> _"Find the SIREN of LVMH and summarise their latest fiche."_
>
> _"Who are the dirigeants of 552032534, and how are they connected to other
> companies?"_
>
> _"Screen NAF 6201Z companies in Paris by finance quality, then give me the
> three most profitable."_
>
> _"Run a PEP check on the directors of this company."_

## Quick start

### 1. Get an Origami API key

The MCP is public but authenticated per key. Create a free-tier key:

1. Sign in at **<https://origami-entreprises.fr>**
2. Go to **Account → API keys → New key**
   (<https://origami-entreprises.fr/account/api-keys/new/>)
3. Copy the `ori_mcp_…` secret. It is shown only once.

### 2. Install the plugin

```text
/plugin marketplace add anthropics/claude-plugins-community
/plugin install origami@claude-community
```

When the plugin is enabled, Claude Code asks for your Origami API key. Paste the
`ori_mcp_…` secret. It goes into your OS keychain (never into `settings.json`)
and travels only as the `Authorization` header to the MCP endpoint. There is no
`mcp-remote` bridge to install and no environment variable to edit.

<details>
<summary>Run it locally from a checkout (before it's published)</summary>

```bash
git clone https://github.com/gregtap/origami-mcp-plugin
claude --plugin-dir ./origami-mcp-plugin
```

Claude Code asks for the key on first load.

</details>

## Tools

All 16 tools the server exposes:

| Tool | What it does |
|------|--------------|
| `suggestions` | Autocomplete on company name, SIREN, or dirigeant name |
| `search_companies` | BM25 search plus filters (`q`, `naf`, `dept`, `etat`, `sort`) over ~29M legal units |
| `search_companies_by_finance_quality` | Pre-filter by BFR (working-capital) trajectory |
| `search_companies_by_lbo_eligibility` | Pre-filter by LBO shape: turnover, age, profitability, leverage, collective proceedings |
| `top_rentable_companies` | Highest-margin SIRENs by NAF or department |
| `get_company` | Full fiche: identity, siège, establishments, dirigeants, 3-year finance summary, recent BODACC, marques, CAC40 flag |
| `search_dirigeants` | Officers from the current RNE snapshot plus BODACC historical matches |
| `get_dirigeant` | One officer's identity and full mandate list |
| `get_comptes_annuels_kpi` | Multi-year annual-account KPIs for one SIREN |
| `get_comptes_annuels_kpi_batch` | Same KPIs for up to 50 SIRENs in a single call |
| `search_bodacc` | BODACC announcements by SIREN, date range, or famille |
| `get_cartographie` | 2-hop person and company ownership or officer network |
| `get_amf_documents` | AMF regulated-filings archive |
| `get_actes_for_siren` | Navigable table of contents over INPI acte PDFs |
| `pep_check` | HATVP politically-exposed-person check, with homonym disambiguation |
| `get_account` | Your key's tier and rate limit |

The plugin also ships a skill, **`/origami:france-companies`**, that teaches
Claude the SIREN-first workflow, the right tool chaining for each task, and the
data caveats worth surfacing.

## Good to know

- **Rate limits** apply per key, per minute. On a `[rate_limited]` error Claude
  backs off on its own, and `get_account` shows your current limit.
- **Privacy by design.** Dirigeant dates of birth come back as `YYYY-MM` only,
  and beneficial-ownership (RBE) data is access-gated under INPI's *intérêt
  légitime* rules. Claude is told to surface these limits rather than imply the
  data is complete.
- **Coverage.** Officer (dirigeant) data is most complete for people currently
  in post and can miss past or some current officers, so the plugin tells Claude
  not to present an officer list as the full set.

## Links

- Origami: <https://origami-entreprises.fr>
- Claude Code plugins: <https://code.claude.com/docs/en/plugins>
- Model Context Protocol: <https://modelcontextprotocol.io>

## License

[MIT](./LICENSE) © Holofin
