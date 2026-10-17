# OpenAI Agents with Fireworks

Small demo workspace for calling Fireworks AI through OpenAI-compatible APIs and the OpenAI Agents SDK.

## What's here

- `openai_agents_sdk.py` - OpenAI Agents SDK demo. It creates a `FileLister` agent with a local `list_files` function tool and uses Fireworks for model inference.
- `response_api.py` - Minimal OpenAI-compatible Responses API call against Fireworks.
- `setup_venv.sh` - Helper script that creates `venv312` with Homebrew Python 3.12 and installs `openai-agents`.
- `requirements.txt` - Pinned Python dependencies from the working environment.
- `docs/` - Asteroids game benchmark and showcase comparing client-only implementations generated from a single prompt across different LLMs and coding harnesses (published via GitHub Pages).

## Setup

From this directory:

```bash
./setup_venv.sh
source venv312/bin/activate
```

Or, if you already have a virtual environment:

```bash
python -m pip install -r requirements.txt
```

## Fireworks configuration

All configuration lives in a local `.env` file that is **gitignored** and never committed. A tracked template is provided in `.env.example`.

1. Copy the template:

   ```bash
   cp .env.example .env
   ```

2. Edit `.env` and set your Fireworks API key:

   ```text
   FIREWORKS_BASE_URL=https://api.fireworks.ai/inference/v1
   FIREWORKS_MODEL=accounts/fireworks/models/glm-5p2
   FIREWORKS_API_KEY=your_fireworks_key
   ```

The Python demos load `.env` automatically via `python-dotenv` (already in `requirements.txt`). You can also export the variables in your shell instead of using `.env`:

```bash
export FIREWORKS_API_KEY="your_fireworks_key"
```

Both demos use:

```text
https://api.fireworks.ai/inference/v1
accounts/fireworks/models/glm-5p2
```

## Run the demos

Agents SDK tool-calling demo:

```bash
python openai_agents_sdk.py
```

Simple Responses API demo:

```bash
python response_api.py
```

Asteroids showcase gallery:

```bash
open docs/index.html
# Or directly open the Sonnet 5.5 implementation:
open docs/sonnet55/index.html
```

## Asteroids Benchmark: Model Variations

The `docs/` directory hosts a client-only Asteroids game benchmark comparing code generated from the exact same one-shot prompt across different models and coding harnesses:

> "Let’s implement the classic asteroid game as a web page. I also want to be able to use cursor buttons to turn and move. Space is shooting. You can make the visuals cool, but don’t complicate the game logic itself, it’ll be client only for demo."

### Implementations

| Model | Harness | Inference Provider | Demo File | Highlights |
| :--- | :--- | :--- | :--- | :--- |
| **Anthropic Sonnet 5.5** | [Strands Code](https://github.com/CsabaConsulting/strands-code/) | AWS Bedrock | [`docs/sonnet55/index.html`](docs/sonnet55/index.html) | Single self-contained HTML page, neon glowing vector outlines, drifting starfield, particle explosions, screen shake. Added down-arrow braking. Full run log in [`docs/sonnet55/full_log.txt`](docs/sonnet55/full_log.txt). |
| **GLM 5.2** | Codex | Fireworks.ai | [`docs/glm52/asteroids.html`](docs/glm52/asteroids.html) | Neon vector art, particle explosions, starfield, collision fix added for bullet radius. |
| **Kimi K2.7 Code** | Codex | Fireworks.ai | [`docs/kimi27code/asteroids.html`](docs/kimi27code/asteroids.html) | Green-on-ink acid neon visuals, independent from-scratch implementation. |
| **DeepSeek V4** | Codex | Fireworks.ai | [`docs/deepseekv4pro/asteroids.html`](docs/deepseekv4pro/asteroids.html) | Classic black canvas with cyan UI, includes generation response log (`docs/deepseekv4pro/response.md`). |
| **MiniMax M3** | Codex | Fireworks.ai | `docs/minimaxm3/` | *Pending* |
| **Nemotron 3 Ultra** | Codex | Fireworks.ai | `docs/nemotron3ultra/` | *Pending* |

### Viewing the Demos

You can open the main gallery locally or view it via GitHub Pages:

```bash
open docs/index.html
```

Or open individual implementations directly:

```bash
open docs/sonnet55/index.html
open docs/glm52/asteroids.html
open docs/kimi27code/asteroids.html
open docs/deepseekv4pro/asteroids.html
```

## Notes

The Fireworks model ID contains slashes: `accounts/fireworks/models/glm-5p2`. If that string is passed directly as an Agents SDK model name, the SDK may interpret `accounts` as a provider prefix and raise:

```text
agents.exceptions.UserError: Unknown prefix: accounts
```

`openai_agents_sdk.py` avoids that by constructing an explicit `OpenAIChatCompletionsModel` with an `AsyncOpenAI` client pointed at the Fireworks base URL.

## Security

Do not commit real API keys. If a real key was ever saved in this folder, rotate it in Fireworks.
