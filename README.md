# OpenAI Agents with Fireworks

Small demo workspace for calling Fireworks AI through OpenAI-compatible APIs and the OpenAI Agents SDK.

## What's here

- `openai_agents_sdk.py` - OpenAI Agents SDK demo. It creates a `FileLister` agent with a local `list_files` function tool and uses Fireworks for model inference.
- `response_api.py` - Minimal OpenAI-compatible Responses API call against Fireworks.
- `setup_venv.sh` - Helper script that creates `venv312` with Homebrew Python 3.12 and installs `openai-agents`.
- `requirements.txt` - Pinned Python dependencies from the working environment.
- `index.html` - Standalone tic-tac-toe landing page/demo. Open it directly in a browser.

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

Standalone HTML page:

```bash
open index.html
```

## Notes

The Fireworks model ID contains slashes: `accounts/fireworks/models/glm-5p2`. If that string is passed directly as an Agents SDK model name, the SDK may interpret `accounts` as a provider prefix and raise:

```text
agents.exceptions.UserError: Unknown prefix: accounts
```

`openai_agents_sdk.py` avoids that by constructing an explicit `OpenAIChatCompletionsModel` with an `AsyncOpenAI` client pointed at the Fireworks base URL.

## Security

Do not commit real API keys. If a real key was ever saved in this folder, rotate it in Fireworks.
