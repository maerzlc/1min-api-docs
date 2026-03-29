# 1min.ai API Inventory And Usage

This document was built from live evidence, not just static docs text. The source set used for this inventory was:

- The live docs app in a browser, including JS-populated sections and network requests.
- The live docs bundle route manifest extracted from `docs.1min.ai` JavaScript.
- The live public model enumeration endpoint `GET https://api.1min.ai/models?feature=...`.

Supporting machine-readable artifacts generated during this pass:

- `docs/1min_feature_inventory.json`
- `docs/1min_docs_inventory.json`

## Scope

Confirmed in this pass:

- `184` canonical docs routes exist in the live docs bundle.
- `51` canonical core or feature-family pages were selected for analysis.
- `59` feature enums were confirmed by successful responses from `GET /models?feature=<FEATURE_ENUM>`.

## Confirmed Public Endpoints

| Endpoint | Method | Status in docs/app | Notes |
| --- | --- | --- | --- |
| `https://api.1min.ai/api/chat-with-ai` | `POST` | Confirmed | Non-streaming unified chat |
| `https://api.1min.ai/api/chat-with-ai?isStreaming=true` | `POST` | Confirmed | SSE streaming unified chat |
| `https://api.1min.ai/api/features` | `POST` | Confirmed | Non-chat feature execution |
| `https://api.1min.ai/api/features?isStreaming=true` | `POST` | Confirmed | Legacy streaming chat via AI Feature API |
| `https://api.1min.ai/api/assets` | `POST` | Confirmed | Asset upload |
| `https://api.1min.ai/api/conversations` | `POST` | Confirmed | Conversation creation |
| `https://api.1min.ai/models?feature=<FEATURE_ENUM>` | `GET` | Confirmed by live docs network and direct API calls | Undocumented but active model enumeration endpoint |
| `https://api.1min.ai/api/recraft/styles` | `GET` | Confirmed in docs content | Auxiliary endpoint referenced by Recraft background replacement docs |
| `https://api.1min.ai/api/recraft/styles?taskType=recraftv3` | `GET` | Confirmed in docs content | Recraft style filtering variant |

Observed behavior:

- `GET https://api.1min.ai/models` returns `{"models":[],"total":0}` without a `feature` query parameter.
- The docs UI uses `GET /models?feature=...` to populate live model pickers.

## Authentication Notes

There is a docs inconsistency here:

- The intro page says to send `Authorization: Bearer YOUR_API_KEY`.
- The feature-specific docs and live playgrounds consistently show `API-KEY: <api-key>`.

Operationally, the feature docs and live docs playgrounds point to `API-KEY` as the request header convention for the public REST endpoints described here.

## Model Enumeration API

### Request

Use:

```text
GET https://api.1min.ai/models?feature=<FEATURE_ENUM>
```

Example:

```text
GET https://api.1min.ai/models?feature=UNIFY_CHAT_WITH_AI
```

### Response Shape

Observed top-level response keys:

- `models`
- `total`

Observed model object keys:

- `uuid`
- `modelId`
- `name`
- `group`
- `provider`
- `status`
- `cutoffDate`
- `logoUrl`
- `features`
- `creditMetadata`
- `modality`
- `information`
- `createdAt`
- `createdBy`
- `updatedAt`
- `updatedBy`
- `numberOfStars`
- `creditEstimation`

Observed `creditMetadata` keys:

- `INPUT`
- `OUTPUT`
- `CONTEXT`
- `LOW_IMAGE`
- `MAX_OUTPUT_TOKEN`

Observed `modality` keys:

- `INPUT`
- `OUTPUT`

## Recommended Usage Flow

### 1. Discover valid models for a feature

Call the live enumeration endpoint first:

```text
GET /models?feature=IMAGE_GENERATOR
GET /models?feature=TEXT_TO_VIDEO
GET /models?feature=UNIFY_CHAT_WITH_AI
```

### 2. Upload assets when a feature needs files

Upload images, PDFs, audio, or documents with:

```text
POST /api/assets
```

Then reuse the returned asset `key` or `path` in later requests.

### 3. Create a conversation only when needed

Use:

```text
POST /api/conversations
```

This is needed for conversation-based workflows such as:

- `UNIFY_CHAT_WITH_AI`
- `CHAT_WITH_AI`
- `CHAT_WITH_IMAGE`
- `CHAT_WITH_PDF`
- `CHAT_WITH_YOUTUBE_VIDEO`

### 4. Choose the correct execution endpoint

Use `POST /api/chat-with-ai` for unified chat.

Use `POST /api/features` for image, audio, video, writing, code, and legacy chat-style feature types.

## Core Usage Patterns

### Unified chat

```json
{
  "type": "UNIFY_CHAT_WITH_AI",
  "model": "gpt-5-nano",
  "promptObject": {
    "prompt": "Reply with OK only."
  }
}
```

Endpoint:

```text
POST /api/chat-with-ai
```

### Generic feature execution

```json
{
  "type": "IMAGE_GENERATOR",
  "model": "black-forest-labs/flux-schnell",
  "promptObject": {
    "prompt": "a red fox in snow"
  }
}
```

Endpoint:

```text
POST /api/features
```

### Asset upload

```text
POST /api/assets
Content-Type: multipart/form-data
field name: asset
```

### Conversation creation

```json
{
  "type": "UNIFY_CHAT_WITH_AI",
  "title": "My chat session",
  "model": "gpt-5-nano"
}
```

Endpoint:

```text
POST /api/conversations
```

## Confirmed Feature Inventory

The table below lists the `59` feature enums confirmed by successful live `/models` responses.

| Model count | Confirmed feature enums |
| --- | --- |
| `68` | `UNIFY_CHAT_WITH_AI` |
| `61` | `CHAT_WITH_AI` |
| `59` | `KEYWORD_RESEARCH` |
| `58` | `CONTENT_GENERATOR_BLOG_ARTICLE` |
| `56` | `CHAT_WITH_YOUTUBE_VIDEO`, `CONTENT_EXPANDER`, `CONTENT_GENERATOR_BRAND_VOICE`, `CONTENT_GENERATOR_EMAIL`, `CONTENT_GENERATOR_EMAIL_REPLY`, `CONTENT_GENERATOR_FACEBOOK_ADS`, `CONTENT_GENERATOR_FACEBOOK_COMMENT`, `CONTENT_GENERATOR_FACEBOOK_POST`, `CONTENT_GENERATOR_GOOGLE_ADS`, `CONTENT_GENERATOR_INSTAGRAM_POST`, `CONTENT_GENERATOR_LINKEDIN_COMMENT`, `CONTENT_GENERATOR_LINKEDIN_POST`, `CONTENT_GENERATOR_TIKTOK_DESCRIPTION`, `CONTENT_GENERATOR_X_COMMENT`, `CONTENT_GENERATOR_X_TWEET`, `CONTENT_SHORTENER`, `CONTENT_TRANSLATOR`, `GRAMMAR_CHECKER`, `PARAPHRASER`, `REWRITER`, `SUMMARIZER`, `YOUTUBE_SUMMARIZER`, `YOUTUBE_TRANSCRIBER`, `YOUTUBE_TRANSLATOR` |
| `34` | `IMAGE_GENERATOR` |
| `28` | `CHAT_WITH_PDF` |
| `23` | `CHAT_WITH_IMAGE` |
| `20` | `CODE_GENERATOR` |
| `14` | `IMAGE_EDITOR` |
| `12` | `SPEECH_TO_TEXT` |
| `11` | `IMAGE_VARIATOR`, `TEXT_TO_VIDEO` |
| `9` | `IMAGE_TO_VIDEO` |
| `6` | `AUDIO_TRANSLATOR` |
| `5` | `IMAGE_INPAINTER`, `TEXT_TO_SPEECH` |
| `4` | `BACKGROUND_REPLACER`, `MUSIC_GENERATOR` |
| `3` | `BACKGROUND_REMOVER`, `IMAGE_OBJECT_REMOVER`, `IMAGE_TO_PROMPT`, `IMAGE_UPSCALER` |
| `2` | `FACE_SWAPPER`, `IMAGE_3D_GENERATOR`, `IMAGE_EXTENDER`, `SKETCH_TO_IMAGE` |
| `1` | `CAPTIONS_GENERATOR`, `SEARCH_AND_REPLACE`, `TEXT_REMOVER`, `TEXT_TO_SOUND`, `VIDEO_FACE_SWAPPER`, `VOICE_CHANGER`, `VOICE_CLONING`, `VOICE_DESIGN`, `VOICE_ISOLATOR` |

## Family-Level Inventory

### Chat and conversation

| Feature enum | Models | Notes |
| --- | --- | --- |
| `UNIFY_CHAT_WITH_AI` | `68` | Dedicated unified chat endpoint |
| `CHAT_WITH_AI` | `61` | Legacy chat via `POST /api/features` |
| `CHAT_WITH_IMAGE` | `23` | Legacy image chat |
| `CHAT_WITH_PDF` | `28` | Legacy PDF chat |
| `CHAT_WITH_YOUTUBE_VIDEO` | `56` | Legacy YouTube chat |

Conversation-specific finding:

- The live `Conversation API` docs page fetched models for `CHAT_WITH_AI` and `UNIFY_CHAT_WITH_AI`.
- The same page documents `CHAT_WITH_IMAGE`, `CHAT_WITH_PDF`, and `CHAT_WITH_YOUTUBE_VIDEO` conversation types, but those were not fetched into the visible docs model section during browser inspection.

### Writing

| Feature enum | Models |
| --- | --- |
| `CONTENT_EXPANDER` | `56` |
| `CONTENT_SHORTENER` | `56` |
| `CONTENT_TRANSLATOR` | `56` |
| `PARAPHRASER` | `56` |
| `SUMMARIZER` | `56` |
| `GRAMMAR_CHECKER` | `56` |
| `KEYWORD_RESEARCH` | `59` |
| `REWRITER` | `56` |
| `CONTENT_GENERATOR_BLOG_ARTICLE` | `58` |
| `CONTENT_GENERATOR_INSTAGRAM_POST` | `56` |
| `CONTENT_GENERATOR_FACEBOOK_POST` | `56` |
| `CONTENT_GENERATOR_FACEBOOK_ADS` | `56` |
| `CONTENT_GENERATOR_GOOGLE_ADS` | `56` |
| `CONTENT_GENERATOR_TIKTOK_DESCRIPTION` | `56` |
| `CONTENT_GENERATOR_X_TWEET` | `56` |
| `CONTENT_GENERATOR_LINKEDIN_POST` | `56` |
| `CONTENT_GENERATOR_EMAIL` | `56` |
| `CONTENT_GENERATOR_EMAIL_REPLY` | `56` |
| `CONTENT_GENERATOR_LINKEDIN_COMMENT` | `56` |
| `CONTENT_GENERATOR_X_COMMENT` | `56` |
| `CONTENT_GENERATOR_FACEBOOK_COMMENT` | `56` |
| `CONTENT_GENERATOR_BRAND_VOICE` | `56` |

### Code

| Feature enum | Models |
| --- | --- |
| `CODE_GENERATOR` | `20` |

### Audio

| Feature enum | Models |
| --- | --- |
| `AUDIO_TRANSLATOR` | `6` |
| `SPEECH_TO_TEXT` | `12` |
| `TEXT_TO_SOUND` | `1` |
| `TEXT_TO_SPEECH` | `5` |
| `MUSIC_GENERATOR` | `4` |
| `VOICE_CHANGER` | `1` |
| `VOICE_CLONING` | `1` |
| `VOICE_DESIGN` | `1` |
| `VOICE_ISOLATOR` | `1` |

### Image

| Feature enum | Models |
| --- | --- |
| `IMAGE_3D_GENERATOR` | `2` |
| `BACKGROUND_REMOVER` | `3` |
| `BACKGROUND_REPLACER` | `4` |
| `FACE_SWAPPER` | `2` |
| `IMAGE_EXTENDER` | `2` |
| `IMAGE_GENERATOR` | `34` |
| `IMAGE_INPAINTER` | `5` |
| `IMAGE_EDITOR` | `14` |
| `IMAGE_TO_PROMPT` | `3` |
| `IMAGE_UPSCALER` | `3` |
| `IMAGE_VARIATOR` | `11` |
| `IMAGE_OBJECT_REMOVER` | `3` |
| `SEARCH_AND_REPLACE` | `1` |
| `SKETCH_TO_IMAGE` | `2` |
| `TEXT_REMOVER` | `1` |

### Video

| Feature enum | Models |
| --- | --- |
| `CAPTIONS_GENERATOR` | `1` |
| `IMAGE_TO_VIDEO` | `9` |
| `TEXT_TO_VIDEO` | `11` |
| `VIDEO_FACE_SWAPPER` | `1` |
| `YOUTUBE_SUMMARIZER` | `56` |
| `YOUTUBE_TRANSCRIBER` | `56` |
| `YOUTUBE_TRANSLATOR` | `56` |

## Notable Confirmed Type Mappings

Some feature-family landing pages did not expose the exact `type` enum cleanly, but model pages did. Confirmed examples:

| Docs family | Confirmed type enum |
| --- | --- |
| Image Mask Editor | `IMAGE_INPAINTER` |
| Image Text Editor | `IMAGE_EDITOR` |
| Object Remover | `IMAGE_OBJECT_REMOVER` |
| Background Remover | `BACKGROUND_REMOVER` |
| Image Extender | `IMAGE_EXTENDER` |
| Search and Replace | `SEARCH_AND_REPLACE` |
| Text Remover | `TEXT_REMOVER` |

## Practical Notes

- If you need a model picker, do not scrape the rendered docs text alone. Use `GET /models?feature=...`.
- The static docs text often omits the JS-loaded model lists.
- The feature-family landing pages are not consistently enough to reconstruct exact request schemas without either opening model-specific pages or inspecting the live docs browser network.

## Recommended Integration Strategy

If you are building a client or proxy:

1. Treat `GET /models?feature=...` as the source of truth for feature-to-model availability.
2. Use `POST /api/chat-with-ai` for all new chat integrations.
3. Use `POST /api/features` for non-chat features.
4. Use `POST /api/assets` before any image, file, PDF, or audio workflow that requires uploaded assets.
5. Use `POST /api/conversations` only when you need persistent context.

## Raw Evidence Files

The full collected inventory and grouped API results are stored in:

- `docs/1min_feature_inventory.json`
- `docs/1min_docs_inventory.json`
