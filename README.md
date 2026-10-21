# pi-palm-anti

Google Cloud Gemini Enterprise (`businessaicode.googleapis.com`) and Antigravity (`cloudcode-pa.googleapis.com`) provider for [Pi Coding Agent](https://pi.dev), with `gemini-4-argon` (`low`, `medium`, `high`) support.

## Install

```bash
pi remove npm:pi-antigravity
pi install git:github.com/sariskiat/pi-palm-anti
```

`~/.pi/agent/settings.json`:

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

Update to the latest commit:

```bash
git -C ~/.pi/agent/git/github.com/sariskiat/pi-palm-anti pull --ff-only origin main
```

Then run `/reload` in Pi.

## Setup

1. Sign in with Google OAuth:
   ```text
   /login antigravity
   ```
2. Switch between **Gemini Enterprise** and **Consumer Subscription** mode at any time:
   ```text
   /antigravity.project <your-gcp-project-id>
   /antigravity.project subscription
   ```
3. Select a model:
   ```text
   /model antigravity/gemini-4-argon
   ```

## Models

<!-- prettier-ignore -->
| Model | Thinking levels | Enterprise backend | Subscription backend |
| --- | --- | --- | --- |
| `antigravity/gemini-4-argon` | `low`, `medium`, `high` | `businessaicode.googleapis.com` | `cloudcode-pa.googleapis.com` |
| `antigravity/gemini-3.8-flash` | `low`, `medium`, `high` | `businessaicode.googleapis.com` | `cloudcode-pa.googleapis.com` |
| `antigravity/gemini-3.1-pro` | `low`, `high` | `businessaicode.googleapis.com` | `cloudcode-pa.googleapis.com` |
| `antigravity/claude-opus-4-6` | `high` | `cloudcode-pa.googleapis.com` | `cloudcode-pa.googleapis.com` |
| `antigravity/claude-sonnet-4-6` | `high` | `cloudcode-pa.googleapis.com` | `cloudcode-pa.googleapis.com` |
| `antigravity/gpt-oss-120b` | `medium` | `cloudcode-pa.googleapis.com` | `cloudcode-pa.googleapis.com` |

## Commands

<!-- prettier-ignore -->
| Command | Description |
| --- | --- |
| `/login antigravity` | Sign in with Google OAuth |
| `/antigravity.project [subscription \| <gcp-project-id>]` | Show or switch between Enterprise GCP project and Subscription mode |
| `/antigravity.models` | List available models and remaining quota |
| `/antigravity.usage` | Show quota usage and reset times |
| `/antigravity.accounts` | List, `switch <index\|email>`, or `remove <index\|email>` linked Google accounts |
| `/antigravity.refresh` | Refresh the model catalog cache |
| `/antigravity.doctor` | Show active endpoint, project, runtime model, and last request status |
| `/antigravity.search <query>` | Search the web with Google Search Grounding |
| `/antigravity.image <prompt>` | Generate an image into `.pi/generated-images/` |

## License

[MIT](LICENSE)
