# @textile-designer-ai/mcp

MCP server for [Textile Designer AI](https://www.textile-designer.ai). It lets
Claude Code, Claude Desktop, Codex, Cursor or any other MCP client run the
Textile Designer tools on image files: upscale, repeat, background removal,
channeling, colourways, vectorizer and the rest.

It is a thin client of `https://www.textile-designer.ai/api/plugin/*`, the
same API the Photoshop plugins use. You log in with your Textile Designer
account; credits, plan access and enabled tools are decided by the server.
Photoshop is not involved: inputs are image files or URLs, outputs are files.

Requires Node.js 18 or newer.

## About this repository

This repository holds the published build of the MCP server (`dist/index.js`, a single file with no runtime dependencies), exactly as released on npm as [`@textile-designer-ai/mcp`](https://www.npmjs.com/package/@textile-designer-ai/mcp). Run it with `npx -y @textile-designer-ai/mcp`, or `node dist/index.js` from a clone. A `Dockerfile` is included for registries that inspect the server in a container. The Claude plugin bundle lives in [ScientiaAI/textile-designer-ai-claude](https://github.com/ScientiaAI/textile-designer-ai-claude).

Security reports: https://www.textile-designer.ai/contact (subject "Security").

## Install

### Claude Code

```bash
claude mcp add textile-designer-ai -- npx -y @textile-designer-ai/mcp
```

### Claude Desktop

Add to `claude_desktop_config.json` (Settings → Developer → Edit Config):

```json
{
  "mcpServers": {
    "textile-designer-ai": {
      "command": "npx",
      "args": ["-y", "@textile-designer-ai/mcp"]
    }
  }
}
```

### Codex

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.textile_designer_ai]
command = "npx"
args = ["-y", "@textile-designer-ai/mcp"]
```

### Cursor

Add to `.cursor/mcp.json` (project) or `~/.cursor/mcp.json` (global):

```json
{
  "mcpServers": {
    "textile-designer-ai": {
      "command": "npx",
      "args": ["-y", "@textile-designer-ai/mcp"]
    }
  }
}
```

On Windows, if `npx` is not found by the client, use `"command": "npx.cmd"`
or the full path to it.

## Logging in

Ask the assistant to connect, or call the `login` tool. The server starts a
device-code login: it opens `textile-designer.ai/plugin-auth` in your browser
with a short code, you confirm it there, and the server picks up the session
in the background. `whoami` confirms the connection and shows your credits,
plan access and enabled tools. `logout` ends the session.

The session token is stored in `~/.textile-designer/mcp-session.json`
(readable only by you). It is never printed.

Example conversation:

> Connect to Textile Designer, then make a half-drop repeat of
> `C:\designs\paisley.png` at 300 dpi and tell me what it cost.

## Tools

Account and jobs:

| Tool | What it does |
|---|---|
| `login` | Start the device-code login (opens the browser; `wait_seconds` blocks until linked). |
| `whoami` | Email, organisation, credits, plan access, enabled tools. |
| `logout` | End the session and forget the token. |
| `job_status` | Status / progress of a job by `task_id`; `wait_seconds` polls until done. |
| `download_result` | Save a completed job's outputs to disk. |
| `list_tools` | Every design tool with a one-line description and its credit rule. |
| `describe_tool` | One tool's best use, how it compares with similar tools, what its settings change, its outputs, modes and credit rule. |
| `guide` | Answers "which tool should I use", typical workflows (digital, screen or rotary, garment to collection, vector), comparisons, print basics, results, credits and plans. |
| `boost_unlimited` | Truly Unlimited only: use a Boost to double today's budget and start queued slow-lane jobs now. Claude asks you first; it spends a boost. |

Design tools (website names):

| Tool | Website label | Result |
|---|---|---|
| `td_ready_to_print` | Ready to Print | image |
| `td_anti_blur` | Anti-Blur | image |
| `td_super_scaler` | Super Scaler | image |
| `td_background_removal` | Background Removal | image |
| `td_watermark_removal` | Watermark Removal | image |
| `td_repeat_set` | Repeat Set | image |
| `td_design_extension` | Design Extension | image |
| `td_border_outline` | Border Outline | image |
| `td_object_layering` | Object Layering | PSD |
| `td_colour_layering` | Colour Layering | PSD |
| `td_style_transfer` | Style Transfer (library style or `style_image`) | image |
| `td_dress_to_design` | Dress to Design | 1-4 images |
| `td_channeling` | Channeling | multichannel PSD + preview |
| `td_color_transfer` | Color Transfer (needs `palette_image`) | image |
| `td_color_matching_extract_colors` | Color Matching: extract palette | JSON |
| `td_color_matching_suggest` | Color Matching: Suggest Colors (market, palette type) | JSON + ready-made replace sets |
| `td_color_matching_match_reference` | Color Matching: match colours from reference images | JSON + replace sets |
| `td_color_matching_swatch_grid` | Color Matching Auto, step 1 (extract colours first) | grid image |
| `td_color_matching_regenerate` | Color Matching Auto, step 2 | images |
| `td_color_matching_replace` | Color Matching: Replace Color | image(s) |
| `td_vectorizer` | Vectorizer (Photoshop-Ready or Illustrator-Ready) | png / tiff / eps (Photoshop-Ready), svg / pdf / eps (Illustrator-Ready) |
| `td_3d_effect` | 3D Effect | images |
| `td_embroidery_effect` | Embroidery Effect | image |
| `td_fabric_texture_removal` | Fabric Texture Removal | images |
| `td_sketch_to_design` | Sketch to Design | image |
| `td_foil_separation` | Foil Separation | image / eps |
| `td_design_generation` | Design Generation (Image+Text, Image to Image, Inpaint) | image |
| `td_design_creation` | Design Creation (prime / new / old / reference) | images |
| `td_colourways` | Colourways | buyer sheet or images |
| `td_spec_board` | Spec Board | image |

Color Matching (FREE) / `psd_variant` is not available: it runs inside
Photoshop and has no server job.

Every design tool accepts:

- `image`: local path or http(s) URL. PNG, JPEG or WebP, up to 80 MB and
  5000 px per side.
- `output_dir`: where to save. Default: the input image's folder (the current
  directory for URL inputs). Existing files are never overwritten.
- `dpi` (tools with a DPI control on the website): output resolution 72-500.
  Omit to keep the image's own resolution.
- `review_settings`: when you give no mode or settings, the tool asks first:
  the mode, then "use the recommended settings, or adjust them one by one?".
  `true` goes straight to each setting; `false` runs with the given or
  recommended settings (Claude passes it after you chose).
- `unlimited_bundle`: request the Unlimited bundle for this run (Unlimited
  plans only; the server decides).
- `estimate_only`: return the credit estimate without uploading or charging.
- `wait` (default true) and `timeout_seconds` (default 240): wait for the job
  and save the outputs. If the job takes longer, the tool returns the
  `task_id` and status `processing`; use `job_status` then `download_result`.
  `wait: false` returns right after submit.

Tool-specific parameters are exactly the controls the website form shows
(same labels, options, ranges and recommended values), described in each
tool's schema. Creativity is shown as a percentage, as on the website, and
sent as a decimal. Nothing runs until the mode and settings are settled, and
prices are quoted only when you ask (`estimate_only`).

### Truly Unlimited plans

- One image per run: the number-of-outputs question is skipped and 1 is sent
  (Dress to Design All Modes still returns one image per mode).
- Super Scaler 8x is not offered.
- Runs over today's budget are queued in the slow lane: the tool returns at
  once with the expected time; check later with `job_status`. `whoami` shows
  today's usage and boosts left, and `boost_unlimited` starts queued jobs now
  (only after you say yes).

## Modes and costs

Tools with modes (Super Scaler's AI Mode, Dress to Design's extraction
models, Design Creation's variants, and so on) describe every option in their
schema as `value (website label): what it does`, and they refuse to run until
that mode argument is given. When the client supports MCP elicitation (a
form prompt), the server asks you directly with the website default marked
"(Recommended)" and preselected; otherwise the call returns
`status: mode_required` with the menu, nothing is uploaded or charged, and the
assistant is instructed to present the modes as options and let you pick.
Website defaults are documented but never applied silently.

After the mode, the server asks once whether to run with the recommended
settings or adjust them; "adjust" (or `review_settings: true`) then asks each
website control in turn, one question at a time, in the website's order, with
the recommended value (or the value already given) preselected. Declining any
question runs nothing. Without elicitation the result lists the same questions,
numbered, with the recommended answer marked. Two read-only tools back this up:

- `list_tools`: every design tool with a one-line description and its credit
  rule, for "what can you do?".
- `describe_tool`: one tool's modes with what each option does, its settings
  (website controls with ranges and recommended values), its credit
  rule, required inputs and result kind, for "what does Prism Shift do?".

Credit figures in descriptions are the website's price list. The server's
estimate is authoritative: ask "what would this cost?" and the assistant calls
the tool with `estimate_only: true`, which uploads nothing and charges nothing.
After a run, the result carries the credits actually charged.

## Prompts (guided workflows)

The server also publishes three MCP prompts. In Claude Code they appear as
slash commands (`/textile-designer-ai:print_ready_pipeline` and so on); other
clients list them under prompts. Each one fills in a message that walks the
assistant through a fixed sequence, with the rule that every paid step is
estimated and shown to you first, any unsettled mode is asked for, credits
are reported after each step, and steps are chained by the saved file paths.

| Prompt | Arguments | Steps |
|---|---|---|
| `print_ready_pipeline` | `image`, `scale` (2/3/4), `repeat_type`, `screens` (2-35) | Ready to Print -> Repeat Set -> Channeling |
| `garment_to_collection` | `image` (garment photo), `mode` (Dress to Design mode), `colourway_count` (6/12/24/48) | Dress to Design -> Colourways -> Spec Board (garment type asked) |
| `vector_pack` | `image`, `ready_for` (photoshop/illustrator) | Vectorizer (format asked from the chosen set) -> Border Outline |

Every tool also carries MCP annotations: the account, job-status and
catalogue tools are read-only; `download_result` writes files but charges
nothing; every `td_*` tool is marked as a paid, non-idempotent run.

## Interactive previews (MCP Apps)

In hosts that support MCP Apps (claude.ai and Claude Desktop, ChatGPT, VS
Code, Goose, Postman as of the ext-apps 1.7 docs), every `td_*` result renders
as a small panel in the conversation: the saved images, their paths and the
credits charged, or the mode menu when a run still needs a mode. The Color
Matching swatch grid renders as a picker: click the cells you like and the
panel sends the chosen variant numbers back so the assistant can estimate and
run `td_color_matching_regenerate`. The pages are self-contained (no CDN,
no network), follow the host's light / dark theme, and are built into the
package. Hosts without MCP
Apps (Claude Code's terminal, Codex, Cursor) ignore them and get the normal
text result with the first image inline.

## Suggestions

After a run completes and its files are saved, the server may add one tip line
to the result (and `structuredContent.suggestions`):

- **Image Search**, when the input image's folder holds 50 or more designs.
- **Cataloguing**, after the third Dress to Design run in a session.

Each tip appears at most once per server session, never after a failed or
estimate-only call, and never for an account that already has the product
(`/me.products.<product>.active`). If the server could not check Image Search
ownership (`checked: false`) the tip is held back. Servers that do not report
`products` at all get each tip once.

## Results

Files are saved as `<input name>_<tool>[_<n>].<ext>`:

- Images: one file per output. Multi-output tools that deliver a zip are
  unpacked into `<input name>_<tool>_<task id>/`.
- Object / Colour Layering: a `.psd`.
- Channeling: the separation `.psd` (one spot channel per screen) unpacked
  from the bundle, plus `<input name>_channeling_preview.png`.
- Color Matching extract: a `.json` palette; the parsed colours are also in
  the structured result.

The tool result lists the paths, the credits charged and the `task_id`, and
includes the first image inline (when it is small enough) so the assistant
can look at it. Results are also kept in your Library on the website.

## Environment variables

| Variable | Purpose |
|---|---|
| `TD_API_BASE_URL` | API origin. Default `https://www.textile-designer.ai`. |
| `TD_TOKEN` | Use this session token instead of the stored one (scripted agents, CI). |
| `TD_SESSION_FILE` | Where to store the token. Default `~/.textile-designer/mcp-session.json`. |

Set them in the client's MCP config (`env` block) when needed.

