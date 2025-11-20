# Markdown Semi-WYSIWYG (View-Pipeline Design)

Goal: keep Markdown source intact and visible, render a semi-WYSIWYG view (styles, flow, structure) without mutating the buffer, and let plugins drive presentation via a structured, incremental pipeline.

## Pipeline Overview
- **Source buffer**: authoritative text; immutable during rendering.
- **View stream** (core): token/spans over the buffer with offsets/markers (text, newline, overlay spans, virtual text anchors).
- **Transform stages** (plugin-capable): plugins can rewrite the view stream for presentation (e.g., turn soft newlines into spaces), inject styles, or add virtual text, but must return source↔view mappings.
- **Layout** (core): consumes the transformed stream plus layout hints (wrap width, centering/margins, table column guides) to produce display lines/segments.
- **Render** (core): draws display segments; no plugin hooks here.

## Hook Points & APIs
1) **Pre-transform hooks (existing)**: plugins may add overlays/virtual text/markers using current APIs.
2) **View transform hook (new)**:
   - Input: viewport slice of the base view stream (tokens with source offsets/marker anchors).
   - Output: transformed stream + mapping table from view positions back to source offsets (for hit-testing/cursors/selections).
   - Allowed ops: replace/suppress tokens (e.g., newline→space), inject style spans, insert virtual text tokens. Buffer remains untouched.
   - Requirements: deterministic (idempotent per input), incremental (viewport or invalidated range only), bounded time.
   - Failure path: core can fall back to previous transform or base stream.
3) **Layout hints API (core)**:
   - Max/compose width (per buffer/split), centering + side margins tint.
   - Soft-break hint alternative: not needed if plugin rewrites newlines in the view stream; for fallback, accept optional soft-break ranges.
   - Table column guides: optional column offsets to align cells.
   - Exposed to plugins via TS ops; core applies during layout.

## View Stream Shape (conceptual)
- Tokens carry: kind (`Text`, `Newline`, `VirtualText`, `StyleSpanStart/End`, `OverlaySpan`), source offset (byte), optional marker id, and style metadata.
- Base stream is derived from buffer + overlays/virtual text resolved to the viewport.
- Transforms operate on this stream and emit a new stream plus a mapping array: for each view token (or character span), the originating source offset (or `None` for injected virtual text).

## Layout Changes (core)
- Accept a transformed stream and mappings.
- Perform wrapping using compose/max width; center the text column with side margins if the terminal is wider.
- When encountering transformed “newline→space” tokens, they wrap like spaces; true newlines still break unless transformed away.
- Cursor/hit-testing uses the mapping to resolve view positions back to source offsets.
- Render status chip shows mode; margins are tinted if centering is active.

## Mode & Features
- Modes: `Source` (no transforms) and `Compose` (transforms active). Stored per buffer/split; status bar shows mode.
- Styles: bold/italic/strong, link color, inline code bg, header tint/weight, block quote tint, strikethrough, task list checkboxes, autolinks.
- Code blocks: shaded bg + monospace; fences dimmed but visible.
- Flow: compose width with centering/margins; visual-line navigation in Compose; Source keeps logical-line nav.
- Structure: headers tinted; lists/checkboxes aligned; block quotes gutter; tables with header shading/column alignment.
- Line breaks: plugin transforms soft breaks by rewriting `\n` tokens to spaces in the view stream (and updating the mapping). Hard breaks stay when user authored (two spaces, backslash+newline, `<br>`) or when outside transform rules. Buffer newlines are untouched.

## Core vs. Plugin Responsibilities
- **Core**: pipeline orchestration; view stream construction; layout (wrap, center, margins, column guides); render; per-buffer mode flag; compose width setting; status chip; mapping-aware cursor/hit-testing; TS ops to set layout hints and submit transformed streams.
- **Plugin (`markdown_compose`)**: parse Markdown incrementally; produce transforms (newline→space inside paragraphs, apply style spans, inject table guides, list bullet fixes); manage compose width preference; bind visual-line navigation in Compose; expose commands (toggle mode, set compose width, reflow on request if desired).

## Implementation Steps
1) Core state: add view mode, compose width, and centering/margin flags to per-split/buffer state; show mode in status bar.
2) View stream builder: create a base token stream per viewport (text/newlines + overlay/virtual text spans with source offsets/markers).
3) TS ops: allow plugin to submit transformed stream+mapping for a viewport; optionally set layout hints (max width, center, column guides).
4) Layout: consume transformed stream, wrap to compose width, center if wider terminal, render margins tint; use mapping for cursor/hit testing.
5) Plugin: implement transform hook (incremental), Markdown parsing, soft-break rewrite rules, style spans, table guides, list alignment, code block styling. Ensure mappings preserve source offsets.

## Notes
- Performance: keep transforms viewport-scoped; reuse previous stream on failure; avoid full-buffer rescans.
- Ordering: single transform hook per buffer per frame; if multiple plugins are needed, define an ordered list and merge mappings deterministically.
- Safety: buffer never changes during rendering; all mutations are opt-in commands (e.g., explicit reflow).***
