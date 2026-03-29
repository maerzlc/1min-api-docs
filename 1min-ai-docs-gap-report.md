# 1min.ai Docs Gap Report

This report compares what is discoverable from the live public API and live docs application versus what is explicitly documented in the 1min.ai docs pages themselves.

## Executive Summary

The current docs are usable for basic manual browsing, but they are not sufficient as a complete implementation reference without browser inspection.

Confirmed during this pass:

- The live docs bundle contains `184` canonical docs routes.
- The live public API supports at least `59` feature enums with working model enumeration through `GET /models?feature=...`.
- The public docs do not document `GET /models?feature=...` as an API surface, even though the live docs UI depends on it.

## High-Impact Gaps

| Gap | What is missing | Evidence | Impact |
| --- | --- | --- | --- |
| Undocumented model enumeration API | No central docs page for `GET /models?feature=<FEATURE_ENUM>` | Live docs network requests call it directly and it returns valid structured JSON | Hard to build dynamic clients or validate feature/model compatibility programmatically |
| Auth header inconsistency | Intro page says `Authorization: Bearer`, feature pages show `API-KEY` | Live docs pages and playgrounds consistently present `API-KEY` | High risk of integration errors |
| Missing machine-readable feature catalog | No single published list of feature enums | `59` feature enums were only confirmed by probing the live `/models` endpoint and drilling into feature/model pages | Makes automated integration planning much harder than necessary |
| JS-only model lists | Static page copies often show `Loading available models...` or omit the model list entirely | Browser inspection reveals the lists are loaded client-side | Static analysis of docs is incomplete |
| Tag pages omit request schema | Many family landing pages show cards but not the exact `type`, shared request schema, or full response shape | Example families include Speech to Text, Music Generator, Text to Sound, Image Mask Editor, Image Text Editor | Developers must click through multiple model pages or inspect browser traffic |
| Conversation page only partially populates models | Conversation docs page fetched models for `CHAT_WITH_AI` and `UNIFY_CHAT_WITH_AI`, not every documented conversation type | Live browser network on the docs page | Available-model coverage on the docs page is incomplete for conversation workflows |

## Detailed Missing Documentation

### 1. Model enumeration endpoint should be documented

Missing:

- Endpoint definition for `GET https://api.1min.ai/models?feature=<FEATURE_ENUM>`
- Query parameter contract for `feature`
- Response schema for `models[]`
- Behavior when `feature` is omitted

Observed:

- `GET /models?feature=UNIFY_CHAT_WITH_AI` returns `68` models.
- `GET /models?feature=IMAGE_GENERATOR` returns `34` models.
- `GET /models?feature=SPEECH_TO_TEXT` returns `12` models.
- `GET /models` returns an empty list.

Recommended doc addition:

- A dedicated `Models API` page.
- One canonical response schema.
- A table of valid `feature` enum values.

### 2. Authentication guidance is internally inconsistent

Current docs problem:

- The intro page instructs clients to use `Authorization: Bearer YOUR_API_KEY`.
- Feature-specific pages and live playgrounds use `API-KEY`.

Why this matters:

- This is the first thing an integrator sees.
- Header mismatches create avoidable authentication failures.

Recommended doc change:

- Choose one canonical auth header convention for the public API docs.
- If both are supported, explicitly document both and say which is preferred.

### 3. Feature-family landing pages often do not contain enough implementation detail

Examples:

- `Speech to Text`
- `Music Generator`
- `Text to Sound`
- `Image Mask Editor`
- `Image Text Editor`
- `Background Remover`
- `Object Remover`

What is missing from those family pages:

- Exact `type` enum
- Canonical shared request schema
- Canonical response schema
- Common constraints and defaults
- The feature-wide model count
- The exact live model IDs

What had to be done instead:

- Open model-specific pages one by one
- Or probe `GET /models?feature=...`

Recommended doc change:

- Every family landing page should include:
  - endpoint
  - auth header
  - exact `type`
  - shared `promptObject` schema
  - one response example
  - live model list

### 4. Important feature enums are only discoverable indirectly

Examples confirmed from model pages, not cleanly from family landing pages:

| Docs family | Confirmed type enum |
| --- | --- |
| Image Mask Editor | `IMAGE_INPAINTER` |
| Image Text Editor | `IMAGE_EDITOR` |
| Object Remover | `IMAGE_OBJECT_REMOVER` |
| Search and Replace | `SEARCH_AND_REPLACE` |
| Text Remover | `TEXT_REMOVER` |

Why this matters:

- A developer landing on the family page cannot safely construct requests from that page alone.

Recommended doc change:

- Put the type enum on the family landing page header block.

### 5. Conversation API docs do not fully explain model availability behavior

Observed:

- The `Conversation API` page documents these supported conversation types:
  - `UNIFY_CHAT_WITH_AI`
  - `CHAT_WITH_AI`
  - `CHAT_WITH_IMAGE`
  - `CHAT_WITH_PDF`
  - `CHAT_WITH_YOUTUBE_VIDEO`
- The live docs browser page fetched model lists for:
  - `CHAT_WITH_AI`
  - `UNIFY_CHAT_WITH_AI`

What is missing:

- Whether model selection for the other conversation types comes from the same `/models` endpoint.
- Whether those types have separate model sets.
- Whether the available-model UI is intentionally partial or just incomplete.

Recommended doc change:

- Explicitly document how available models are resolved for every conversation type.

### 6. The AI Feature API docs defer too much to browser inspection

Current statement:

- The docs explicitly tell users to use DevTools or inspect the web app for detailed request and response examples.

Why this is a problem:

- That is useful as a fallback.
- It should not be necessary for core request construction.

Recommended doc change:

- Replace the “inspect DevTools” fallback with first-class reference pages or keep it only as an advanced note.

### 7. The docs do not publish a complete endpoint index

Confirmed public or public-facing endpoints observed in this pass:

- `POST /api/chat-with-ai`
- `POST /api/chat-with-ai?isStreaming=true`
- `POST /api/features`
- `POST /api/features?isStreaming=true`
- `POST /api/assets`
- `POST /api/conversations`
- `GET /models?feature=<FEATURE_ENUM>`
- `GET /api/recraft/styles`
- `GET /api/recraft/styles?taskType=recraftv3`

What is missing:

- A central endpoint matrix listing all supported public endpoints and which pages explain them.

### 8. Static docs exports are materially incomplete

Observed:

- Plain fetched page text often omits the live model list.
- Several sections render as:
  - `Loading available models...`
  - `Loading interactive playground...`

Why this matters:

- Static fetches, LLM retrieval systems, and search indexing do not see the same information a browser sees.

Recommended doc change:

- Render a server-side fallback table for models and example requests.
- Or publish an official machine-readable JSON index.

### 9. Docs discoverability is weaker than the live docs bundle suggests

Observed:

- The live docs bundle exposes `184` canonical routes.
- The static intro page directly exposes only a much smaller subset.

Impact:

- Important subpages are effectively hidden unless users already know the route structure or browse the sidebar extensively.

Recommended doc change:

- Publish a sitemap or feature index page by category.
- Include model-family landing pages and per-model pages in that index.

### 10. Auxiliary APIs are not indexed well enough

Observed:

- Recraft pages expose `GET /api/recraft/styles` and `GET /api/recraft/styles?taskType=recraftv3`.

What is missing:

- A central description of when these auxiliary endpoints are required.
- Shared schema for their responses.

Recommended doc change:

- Add an “Auxiliary Endpoints” section or separate page.

## Recommended Minimum Additions To The Official Docs

The docs should add, at minimum:

1. A `Models API` page for `GET /models?feature=...`.
2. A canonical feature enum matrix covering all currently supported features.
3. A central endpoint matrix covering all public endpoints.
4. A consistent authentication section used by every API page.
5. Family landing pages that include the exact `type` enum and request schema.
6. A published machine-readable feature inventory, ideally JSON.
7. Server-rendered model tables so static fetches do not lose critical data.

## Suggested Diff Structure For 1min.ai Docs

If the docs team wanted a concrete patch plan, this is the smallest high-value set of additions:

### Add a new `Models API` page

Include:

- `GET /models?feature=<FEATURE_ENUM>`
- parameter docs
- full response schema
- examples
- feature enum table

### Update the intro page

Include:

- one unambiguous auth header convention
- links to `Models API`
- central endpoint index

### Update each feature-family landing page

Include:

- exact `type`
- shared `promptObject` schema
- live model IDs
- response example

### Update the Conversation API page

Include:

- exact model source for every supported conversation type
- explanation of why the page currently loads only certain model lists

## Bottom Line

The public docs are close to being a browsable catalog, but they are not yet a complete implementation reference. The live browser experience and the live `/models` endpoint contain material information that should be documented explicitly.
