# pi-palm-anti

`pi-palm-anti` is a [Pi Coding Agent](https://pi.dev) provider extension for Google Cloud Gemini Enterprise (`businessaicode.googleapis.com`) and Google Antigravity (`cloudcode-pa.googleapis.com`). It supports `gemini-4-argon` (`low`, `medium`, and `high`), `gemini-3.8-flash`, `gemini-3.1-pro`, `claude-sonnet-4-6`, `claude-opus-4-6`, and `gpt-oss-120b` directly inside Pi without calling an external CLI binary.

## Contents

- [Features](#features)
- [Requirements](#requirements)
- [Install](#install)
- [Quick start](#quick-start)
- [Enterprise and consumer routing](#enterprise-and-consumer-routing)
- [Models](#models)
- [Commands](#commands)
- [Configuration](#configuration)
- [Development](#development)
- [License](#license)

## Features

- **Google Cloud Gemini Enterprise support.** Authenticates against `https://businessaicode.googleapis.com`, calls `v1beta:fetchLicenses` to resolve your license tier (such as `gcp-ge-standard-tier`) and serving location (`global`), and streams responses from `v1beta/projects/{project}/locations/{location}:streamGenerateContent`.
- **Gemini 4 Argon support.** Exposes `antigravity/gemini-4-argon` with `low`, `medium`, and `high` reasoning levels (`gemini-4-argon-low`, `gemini-4-argon-medium`, and `gemini-4-argon-high` on Gemini Enterprise).
- **Hybrid model routing.** Routes Gemini models through Gemini Enterprise when an Enterprise GCP project is configured, while routing Claude (`claude-sonnet-4-6`, `claude-opus-4-6`) and GPT-OSS (`gpt-oss-120b`) through Cloud Code Assist (`cloudcode-pa.googleapis.com`).
- **Gemini 4 Argon turn boundary protection.** Keeps `functionResponse` tool-result turns separate from subsequent user text turns so multi-turn tool sessions with mid-turn user steering never fail with HTTP 400 message boundary errors.
- **Multi-account management and automatic failover.** Stores linked accounts in `~/.pi/agent/antigravity-accounts.json` and switches to the next account when a hard quota limit is reached.
- **Built-in web search and image generation.** Registers `google_search` and `generate_image` tools and the `/antigravity.search` and `/antigravity.image` commands.

## Requirements

- Node.js `>=22.15.0`
- `@earendil-works/pi-coding-agent` and `@earendil-works/pi-ai` `>=0.80.0`
- A Google account with access to Gemini Enterprise (`businessaicode.googleapis.com`), Cloud Code Assist (`cloudcode-pa.googleapis.com`), or both

## Install

Remove any previous `pi-antigravity` package first so the two extensions do not conflict:

```bash
pi remove npm:pi-antigravity
```

Install `pi-palm-anti` from GitHub:

```bash
pi install git:github.com/sariskiat/pi-palm-anti
```

Verify that `~/.pi/agent/settings.json` references `git:github.com/sariskiat/pi-palm-anti`:

```json
{
  "defaultProvider": "antigravity",
  "defaultModel": "gemini-4-argon",
  "defaultThinkingLevel": "high",
  "packages": [
    "git:github.com/sariskiat/pi-palm-anti"
  ]
}
```

Run `/reload` inside Pi or restart `pi` after installation.

To pull updates later:

```bash
git -C ~/.pi/agent/git/github.com/sariskiat/pi-palm-anti pull --ff-only origin main
```

## Quick start

1. Start Pi and run `/login antigravity`.
2. Complete Google sign-in in your browser.
3. When prompted for a Google Cloud Project ID:
   - Enter your Gemini Enterprise GCP project ID (for example `<your-gcp-project-id>`) to enable Gemini Enterprise (`businessaicode.googleapis.com`).
   - Leave it blank if you only use a consumer Antigravity account.
4. Select `gemini-4-argon`:

   ```text
   /model antigravity/gemini-4-argon
   ```

5. Check active routing and endpoint status at any time with:

   ```text
   /antigravity.doctor
   ```

To run a single prompt from the shell with `gemini-4-argon` at high reasoning:

```bash
pi -p --model antigravity/gemini-4-argon:high "Explain how this repository is structured"
```

## Enterprise and consumer routing

`pi-palm-anti` supports two Google backends under the `antigravity` provider:

1. **Gemini Enterprise (`https://businessaicode.googleapis.com`).** Used for all `gemini-*` models when your credentials include an Enterprise GCP `projectId` (`authMode: "enterprise"`) or when `cloudcode-pa.googleapis.com` returns license error `#3501`.
   - License discovery calls `POST https://businessaicode.googleapis.com/v1beta:fetchLicenses` with `projectId` and reads `userTier` and `location` from the `GOOGLE_CLOUD_PLATFORM` license entry.
   - Streaming generation calls `POST https://businessaicode.googleapis.com/v1beta/projects/{projectId}/locations/{location}:streamGenerateContent?alt=sse` with `entitlement: { userTier }`.
   - Model catalog and quota checks merge models from both `businessaicode.googleapis.com` and `cloudcode-pa.googleapis.com` and always include `gemini-4-argon`, `gemini-3.1-pro`, and `gemini-3.8-flash` for Enterprise accounts.
2. **Cloud Code Assist (`https://daily-cloudcode-pa.googleapis.com` and `https://cloudcode-pa.googleapis.com`).** Used for consumer accounts and for non-Gemini Vertex models (`claude-sonnet-4-6`, `claude-opus-4-6`, `gpt-oss-120b`) on Enterprise accounts via `v1internal:loadCodeAssist` and `v1internal:streamGenerateContent?alt=sse`.

## Models

<!-- prettier-ignore -->
| Public model ID | Input | Thinking levels | Max output tokens | Enterprise (`businessaicode.googleapis.com`) | Consumer (`cloudcode-pa.googleapis.com`) |
| --- | --- | --- | --- | --- | --- |
| `gemini-4-argon` | Text, image | Low, Medium, High | 65,536 | `gemini-4-argon-low`, `gemini-4-argon-medium`, `gemini-4-argon-high` | `gemini-4-argon` (`thinkingLevel`: `LOW`, `MEDIUM`, `HIGH`) |
| `gemini-3.8-flash` | Text, image | Low, Medium, High | 65,536 | `gemini-3.8-flash-low`, `gemini-3.8-flash-medium`, `gemini-3.8-flash-high` | `gemini-3.8-flash-low`, `gemini-3.8-flash-medium`, `gemini-3.8-flash-high` |
| `gemini-3.7-flash` | Text, image | Low, Medium, High | 65,536 | `gemini-3.7-flash-low`, `gemini-3.7-flash-medium`, `gemini-3.7-flash-high` | `gemini-3.7-flash-low`, `gemini-3.7-flash-medium`, `gemini-3.7-flash-high` |
| `gemini-3.6-flash` | Text, image | Low, Medium, High | 65,536 | `gemini-3.6-flash-low`, `gemini-3.6-flash-medium`, `gemini-3.6-flash-high` | `gemini-3.6-flash-low`, `gemini-3.6-flash-medium`, `gemini-3.6-flash-high` |
| `gemini-3.5-flash` | Text, image | Low, Medium, High | 65,536 | `gemini-3.6-flash-low`, `gemini-3.6-flash-high` | `gemini-3.5-flash-extra-low`, `gemini-3.5-flash-low`, `gemini-3-flash-agent` |
| `gemini-3.1-pro` | Text, image | Low, High | 65,535 | `gemini-3.1-pro-low`, `gemini-3.1-pro-high` | `gemini-3.1-pro-low`, `gemini-pro-agent` |
| `claude-sonnet-4-6` | Text, image | High | 64,000 | Routed via Cloud Code Assist (`claude-sonnet-4-6`) | `claude-sonnet-4-6` |
| `claude-opus-4-6` | Text, image | High | 64,000 | Routed via Cloud Code Assist (`claude-opus-4-6-thinking`) | `claude-opus-4-6-thinking` |
| `gpt-oss-120b` | Text | Medium | 32,768 | Routed via Cloud Code Assist (`gpt-oss-120b-medium`) | `gpt-oss-120b-medium` |

## Commands

<!-- prettier-ignore -->
| Command | Description |
| --- | --- |
| `/login antigravity` | Sign in with Google OAuth and optionally set a Gemini Enterprise GCP project ID. |
| `/model antigravity/<model-id>` | Switch to a model such as `antigravity/gemini-4-argon`. |
| `/antigravity.models` | List available models, capabilities, and remaining quota. |
| `/antigravity.models all` | Include hidden tab and chat models in the list. |
| `/antigravity.usage` | Show quota usage and reset times. |
| `/antigravity.accounts` | List, switch, or remove linked Google accounts (`switch <index\|email>`, `remove <index\|email>`). |
| `/antigravity.refresh` | Force a live refresh of the model catalog and write it to `~/.pi/agent/antigravity-catalog-cache.json`. |
| `/antigravity.doctor` | Display sanitized diagnostics for the last request (`endpoint`, `project`, `runtimeModel`, `status`). |
| `/antigravity.search <query>` | Run a web search with Google Search Grounding (`--thinking`, `--url <url>`). |
| `/antigravity.image <prompt>` | Generate an image into `.pi/generated-images/` (`--ratio 16:9`, `--model`, `--path`). |

## Configuration

Environment variables use the `ANTIGRAVITY_` prefix:

<!-- prettier-ignore -->
| Variable | Purpose |
| --- | --- |
| `ANTIGRAVITY_PROJECT_ID` | Override the Google Cloud project ID used for Gemini Enterprise or Cloud Code Assist requests. |
| `ANTIGRAVITY_BASE_URL` | Override the consumer Cloud Code Assist base URL (must use HTTPS on an allowed `*.googleapis.com` host). |
| `ANTIGRAVITY_RUNTIME_MODEL` | Pin requests to a specific runtime model ID such as `gemini-4-argon-high`. |
| `ANTIGRAVITY_USER_AGENT` | Override the `User-Agent` header sent to Google APIs. |
| `ANTIGRAVITY_CATALOG_REFRESH_INTERVAL_MS` | Override the 4-hour catalog cache TTL in milliseconds (`14400000` by default). |
| `ANTIGRAVITY_CALLBACK_HOST` | Bind the local OAuth callback server to `127.0.0.1`, `::1`, or `localhost`. |
| `ANTIGRAVITY_CLIENT_ID` | Override the OAuth client ID. |
| `ANTIGRAVITY_CLIENT_SECRET` | Override the OAuth client secret. |
| `ANTIGRAVITY_NO_KEEPALIVE` | Set to `1` to disable the keep-alive connection pool. |
| `ANTIGRAVITY_NO_PREWARM` | Set to `1` to skip TLS connection pre-warming. |
| `ANTIGRAVITY_NO_EXTRA_TOOLS` | Set to `1` to disable both `google_search` and `generate_image` model tools. |
| `ANTIGRAVITY_NO_SEARCH_TOOL` | Set to `1` to disable only the `google_search` model tool. |
| `ANTIGRAVITY_NO_IMAGE_TOOL` | Set to `1` to disable only the `generate_image` model tool. |
| `ANTIGRAVITY_DEBUG_DUMP` | Set to `1` to write failed request payloads to `/tmp/antigravity-last-request.json`. |

### Credential files

- `~/.pi/agent/auth.json`: Stores the active `antigravity` OAuth credentials (`access`, `refresh`, `expires`, `projectId`, `userTier`, `location`, `authMode`).
- `~/.pi/agent/antigravity-accounts.json`: Stores all linked accounts (`0600` permissions) for `/antigravity.accounts` and quota failover.
- `~/.pi/agent/antigravity-catalog-cache.json`: Stores the cached model catalog and routing table.

Never commit or share `~/.pi/agent/auth.json` or `~/.pi/agent/antigravity-accounts.json`.

## Development

This repository uses Yarn 4 through Corepack and runs TypeScript tests directly on Node.js:

```bash
corepack enable
yarn install
yarn check
```

Run individual checks with:

```bash
yarn typecheck
yarn lint
yarn format:check
yarn security-check
yarn test
```

## License

[MIT](LICENSE)
