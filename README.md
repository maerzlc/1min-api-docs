# 1min.ai API Documentation (Unofficial)

Supplementary documentation for the [1min.ai](https://1min.ai) REST API, compiled through systematic reverse engineering of the live docs application, network traffic analysis, and direct API probing.

This is **not** official documentation. It exists because the official docs, while usable for basic browsing, are not sufficient as a complete implementation reference without browser inspection.

## Why This Exists

The official 1min.ai documentation has several gaps that make programmatic integration difficult:

- **The model enumeration API (`GET /models?feature=...`) is undocumented**, even though the live docs UI depends on it.
- **Authentication headers are inconsistent** -- the intro page says `Authorization: Bearer`, while feature pages use `API-KEY`.
- **Model lists are loaded client-side via JavaScript**, so static fetches and LLM retrieval systems see "Loading available models..." instead of actual data.
- **Feature-family landing pages often omit the exact `type` enum**, request schema, and response shape.
- **There is no published endpoint index or machine-readable feature catalog.**

Full details are in the [gap report](1min-ai-docs-gap-report.md).

## Contents

| File | Description |
|---|---|
| [`1min-ai-api-inventory-and-usage.md`](1min-ai-api-inventory-and-usage.md) | Comprehensive API reference: confirmed endpoints, authentication, model enumeration API, usage patterns, and a full inventory of all 59 confirmed feature enums with model counts |
| [`1min-ai-docs-gap-report.md`](1min-ai-docs-gap-report.md) | Detailed gap analysis with 10 specific issues, evidence, impact assessment, and concrete recommendations for the docs team |
| [`1min_feature_inventory.json`](1min_feature_inventory.json) | Machine-readable inventory of all feature enums and their associated models (~3 MB) |
| [`1min_docs_inventory.json`](1min_docs_inventory.json) | Machine-readable inventory of the 184 canonical docs routes |

## Key Findings

- **184** canonical docs routes exist in the live docs bundle.
- **59** feature enums confirmed via `GET /models?feature=<FEATURE_ENUM>`.
- **68** models available for `UNIFY_CHAT_WITH_AI` (the primary unified chat feature).
- The undocumented `GET /models?feature=...` endpoint is the most reliable way to discover available models.
- `POST /api/chat-with-ai` is the current unified chat endpoint; `POST /api/features` handles non-chat features.

## Methodology

This inventory was built from live evidence, not static docs text:

1. The live docs app at `docs.1min.ai` was inspected in a browser, including JS-populated sections and network requests.
2. The docs bundle route manifest was extracted from the site's JavaScript.
3. All feature enums were systematically probed against `GET https://api.1min.ai/models?feature=...` to confirm which are active and how many models each supports.

## For the 1min.ai Team

If you're from 1min.ai -- this is intended constructively. The gap report includes a concrete "suggested diff structure" section with the smallest high-value set of additions that would make the official docs a complete implementation reference. The recommended minimum additions are:

1. A **Models API** page for `GET /models?feature=...`
2. A canonical **feature enum matrix**
3. A central **endpoint index**
4. A consistent **authentication section**
5. Family landing pages with the exact **`type` enum and request schema**
6. A published **machine-readable feature inventory**
7. **Server-rendered model tables** so static fetches don't lose data

## Related

- [1min-proxy](https://github.com/maerzlc/1min-proxy) -- an OpenAI-compatible API proxy for 1min.ai, built using the knowledge from this research.

## License

This documentation is provided as-is for informational purposes. All referenced APIs and trademarks belong to [1min.ai](https://1min.ai).
