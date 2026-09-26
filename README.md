# ComfyUI-Llama-YuE2

A small ComfyUI custom node that uses an OpenAI-compatible LLM endpoint to turn a plain-language song idea into three coordinated outputs:

- complete, structured YuE2 lyrics
- a detailed YuE2 musical style description
- a diffusion-ready square album-cover prompt

The node includes an opinionated songwriting system prompt designed to produce specific, singable songs rather than generic AI lyrics. It keeps musical and production directions separate from sung lyrics, supports selectable song lengths, generates matching cover-art concepts, and uses a strict JSON schema to keep all three outputs reliable.

## Installation

Clone this repository into `ComfyUI/custom_nodes`:

```bash
cd /path/to/ComfyUI/custom_nodes
git clone <repository-url> ComfyUI-Llama-YuE2
```

Restart ComfyUI.

The node appears as:

**Llama.cpp → YuE2 Song Writer**

under:

**audio/YuE2**

No additional Python packages are required. The node uses Python's standard library.

## LLM server

Run llama.cpp, or another compatible server, with the OpenAI chat-completions API enabled.

Enter the server root in the node, for example:

```text
http://127.0.0.1:8080
```

The node automatically appends:

```text
/v1/chat/completions
```

The server must support JSON-schema response formatting.

## Inputs

| Input | Description |
| --- | --- |
| `prompt` | Plain-language description of the song you want. |
| `song_length` | Requested composition length: `very_short`, `short`, `standard`, or `full`. |
| `llama_url` | Root URL of the OpenAI-compatible LLM server. |
| `temperature` | Creativity setting for generation. Default: `0.8`. |
| `max_tokens` | Maximum LLM response length. Default: `4096`. |
| `seed` | LLM seed. Use `-1` for no fixed seed. |
| `api_key` | Optional API key. When provided, it is sent as a Bearer token. |

## Song length

The `song_length` input changes the songwriting instructions sent to the LLM rather than simply truncating its output.

Available settings:

| Setting | Target length | Approximate lyric budget |
| --- | --- | --- |
| `very_short` | 30–60 seconds | 6–10 sung lines |
| `short` | 60–90 seconds | 10–16 sung lines |
| `standard` | 2–3 minutes | 20–32 sung lines |
| `full` | 3–4 minutes | 32–48 sung lines |

Repeated lyrics and repeated choruses count toward these budgets. Section markers do not.

For `very_short` and `short` songs, the generated style also tells YuE2 to begin vocals quickly and avoid wasting a significant portion of the requested duration on a long instrumental introduction.

YuE2's own maximum-duration setting remains separate and can still be used as a safety ceiling.

## Outputs

| Output | Connect to |
| --- | --- |
| `lyrics` | YuE2 Lyrics input |
| `style` | YuE2 Style input |
| `cover_art` | An image-generation workflow or diffusion prompt input |

### Lyrics

`lyrics` contains complete YuE2-ready song lyrics with section markers such as:

```text
[Verse 1]
[Chorus]
[Bridge]
[Instrumental]
[Outro]
```

Only words intended to be sung are included beneath the section markers.

Musical directions, instrument descriptions, production notes, stage directions, BPM information, and similar annotations are deliberately excluded.

For example, an instrumental passage is represented as:

```text
[Instrumental]
```

rather than:

```text
[Instrumental]
(Guitar solo)
```

Parentheses are also prohibited from the generated lyrics because YuE2 may interpret their contents as vocals.

### Style

`style` contains a compact, comma-separated musical conditioning description for YuE2.

Depending on the song, it can specify characteristics such as:

- language and vocal presentation
- genre and subgenre
- vocal character and delivery
- BPM and tempo feel
- groove and meter
- instrumentation
- arrangement
- section dynamics
- backing vocals
- production and recording character
- mix character
- overall mood

Musical directions that do not belong in the sung lyrics are placed here instead.

Example:

```text
English, warm male folk vocal, upbeat indie folk, 115 BPM, bouncy shuffle, bright acoustic guitar, playful fiddle, upright bass, brushed snare, communal chorus harmonies, conversational delivery, organic live-room production, cheerful nostalgic mood
```

### Cover art

`cover_art` is a standalone diffusion prompt for square album artwork inspired by the generated song.

The prompt attempts to preserve the song's specific characters, objects, setting, humor, mood, era, and musical identity rather than producing generic music artwork.

It describes a coherent square composition including useful details such as:

- subject appearance
- setting
- composition
- lighting
- atmosphere
- visual or photographic style
- relevant objects and premise-specific details

The generator also invents a fictitious album title and band or artist name appropriate to the song and includes explicit typography instructions for placing them on the cover.

The cover-art prompt asks the image model to render only the actual album title and performer name, without labels such as `Album:`, `Title:`, `Band:`, or `Artist:`.

Because the artwork is treated as the cover of an album containing the song rather than necessarily a literal illustration of the song itself, it can remain thematically connected without simply depicting every event in the lyrics.

## Usage

1. Enter a song concept in `prompt`.
2. Choose the desired `song_length`.
3. Set the LLM server URL and optional API key.
4. Connect `lyrics` and `style` to the corresponding YuE2 inputs.
5. Optionally connect `cover_art` to your image-generation workflow.
6. Queue the workflow.

Example prompt:

```text
A short, upbeat folk song about the same group of friends sharing the same dumb jokes in the same online chat room for thirty years.
```

The hidden system prompt handles the detailed songwriting work, including structure, specificity, tone, rhyme, meter, instrumentation, production details, YuE2 formatting, and the matching cover-art concept.

## Songwriting behavior

The system prompt is deliberately opinionated.

It encourages the LLM to identify what is actually distinctive about the user's premise before writing and to build the song around concrete details rather than generic emotional language.

Among other things, it attempts to:

- write lyrics that are naturally singable rather than prose broken into short lines
- preserve specific details from the user's premise
- choose song structure according to the genre rather than mechanically using the same verse/chorus template
- keep line length and lyrical density appropriate for the selected tempo and style
- use rhyme intentionally without forcing every line to rhyme
- create memorable hooks and useful repetition
- make humorous songs contain actual jokes, observations, escalation, callbacks, or other comedy
- avoid inventing tragedy, nostalgia, profundity, or inspirational conclusions when the premise does not call for them
- avoid generic AI-song imagery and stock phrases unless they genuinely fit the song
- keep instrumental, arrangement, performance, and production instructions out of the lyrics
- keep the generated lyrics, musical style, and cover artwork conceptually consistent with one another

When the user's prompt specifies things such as language, genre, vocalist, tempo, instruments, viewpoint, characters, setting, story, or phrases to include, the system prompt instructs the LLM to preserve those choices wherever possible.

When details are omitted, the LLM is expected to make reasonable creative decisions rather than asking follow-up questions.

## Structured output

The LLM is required to return a JSON object containing exactly three non-empty strings:

```json
{
  "lyrics": "...",
  "style": "...",
  "cover_art": "..."
}
```

The request uses a strict JSON schema requiring all three fields and disallowing additional properties.

The node then parses and validates each field before returning it to ComfyUI.

As an additional compatibility measure, it strips a surrounding Markdown code fence if a model ignores the instruction to return raw JSON.

## Notes

- The API key is used only for the outgoing request and is omitted when the field is blank.
- `seed = -1` means no seed is sent to the LLM server.
- The selected song length is added to the user request as a hard compositional constraint.
- Musical and production directions are placed in `style`, not in `lyrics`.
- Purely instrumental sections are represented using section markers such as `[Instrumental]`.
- Parenthetical directions are intentionally prohibited from lyrics.
- Cover artwork is generated as a prompt only; this node does not itself generate the image.
- The cover-art prompt includes a fictitious album title and performer name with typography instructions.
- The node uses a 300-second HTTP timeout for LLM generation.
- If generation fails, ComfyUI reports connection errors, HTTP responses, malformed JSON, unexpected API responses, or missing output fields to help diagnose the server or model response.
- The node requires an OpenAI-compatible server that supports `response_format` with `json_schema`.
