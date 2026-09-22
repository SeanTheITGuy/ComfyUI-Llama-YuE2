# ComfyUI-Llama-YuE2

A small ComfyUI custom node that uses an OpenAI-compatible LLM endpoint to turn a plain-language song idea into two YuE2-ready outputs:

- complete, structured lyrics
- a detailed musical style description

The node includes an opinionated songwriting prompt, separates musical directions from sung lyrics, and uses a strict JSON schema to keep the two outputs reliable.

## Installation

Clone this repository into `ComfyUI/custom_nodes`:

```bash
cd /path/to/ComfyUI/custom_nodes
git clone <repository-url> ComfyUI-Llama-YuE2
```

Restart ComfyUI. The node appears as **Llama.cpp → YuE2 Song Writer** under **audio/YuE2**.

No additional Python packages are required; the node uses Python's standard library.

## LLM server

Run llama.cpp, or another compatible server, with the OpenAI chat-completions API enabled. Enter the server root in the node, for example:

```text
http://127.0.0.1:8080
```

The node appends `/v1/chat/completions` automatically. The server must support JSON-schema response formatting.

## Inputs

| Input | Description |
| --- | --- |
| `prompt` | Plain-language description of the song you want. |
| `llama_url` | Root URL of the OpenAI-compatible LLM server. |
| `temperature` | Creativity setting for lyric generation. Default: `0.8`. |
| `max_tokens` | Maximum response length. Default: `4096`. |
| `seed` | LLM seed. Use `-1` for no fixed seed. |
| `api_key` | Optional API key. When provided, it is sent as a Bearer token. |

## Outputs

| Output | Connect to |
| --- | --- |
| `lyrics` | YuE2 Lyrics input |
| `style` | YuE2 Style input |

## Usage

1. Enter a song concept in `prompt`.
2. Set the LLM server URL and optional API key.
3. Connect `lyrics` and `style` to the corresponding YuE2 inputs.
4. Queue the workflow.

Example prompt:

```text
A short, upbeat folk song about the same group of friends sharing the same dumb jokes in the same online chat room for thirty years.
```

The hidden system prompt handles song structure, specificity, tone, rhyme, meter, instrumentation, production details, and YuE2 formatting.

## Notes

- The API key is used only for the outgoing request and is omitted when the field is blank.
- Musical and production directions are placed in `style`, not in `lyrics`.
- Purely instrumental sections are represented with section markers such as `[Instrumental]`.
- If generation fails, ComfyUI reports the HTTP response or malformed LLM output to help diagnose the server or model response.
