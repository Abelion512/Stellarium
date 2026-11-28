---
name: aero
description: Native interactive transcript cards and UI directives.
version: 0.4.0
author: Abelion
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [ui, visual, interactive, generative-ui, widget]
    category: creative
---

# Aero

## Automatic selection

When a person asks for a visual answer, says they dislike long text, or asks for a procedure, status summary, comparison or short form, choose a fitting card yourself. They should not need to ask for directive syntax. Use prose for simple facts and for any client where native cards are unavailable. Use at most one dominant card by default, plus one short text conclusion.

| Need | kind | Data attributes |
|---|---|---|
| Procedure or checklist | wizard | steps="Review evidence\|Choose an option\|Confirm" |
| Status summary | status | items="Tests:pass\|Live host:not verified" |
| Evidence or work summary | trace | steps="Read source\|Test behavior\|State limits" |
| Compare two options | compare | left="Benefit\|Cost" right="Benefit\|Cost" pick="left" |
| Short non-secret inputs | form | fields="Name\|Notes" submit="Send feedback" |

## Visual understanding, not decorated prose

For people who learn faster by seeing relationships, choose the representation
from the question. Do not ask them to write a directive or name a component.

- Interview or preference question: use `choice`, not an A/B/C wall of text.
  One question at a time, optional note, explicit submit. When a question flow
  is already running, including questions from an invoked interview skill,
  render each question's selectable answers as a `choice` card; keep any
  hypothesis, confidence or guesses as text around the card, one question per
  card. Reuse answers already
  in the conversation instead of repeating the interview. Do not preselect the
  recommended answer or turn a selection into permission for an external action.
- Explain a process or cause-and-effect chain: use `flow`. Put short stage names
  in nodes and explain one stage at a time in details. Clicking a stage is local
  exploration, not a new model call. This is a sequence, not a branching graph.
- Compare quantities: use `chart` only when you have grounded numeric values,
  matching labels and units. Include a short source label. If using fictional
  teaching data, say it is illustrative. Never invent measurements or use bar
  length to imply a proven learning benefit. Use compare/text for qualitative
  differences or unsupported/negative numeric data.

| Need | kind | Data attributes |
|---|---|---|
| Clickable interview | choice | options="Diagram\|Example\|Text" multiple="false" |
| Visual sequence | flow | nodes="Input\|Validate\|Render" details="Understand the ask\|Check allowed data\|Draw native controls" |
| Several linked questions with one decision | interview | groups="Scope\|Trigger" g1_options="Verify::Status only\|Merge::After approval" g2_options="Webhook::On change\|Manual::On command" submit="Save decision" |
| Practice questions with feedback | quiz | questions="Q1" q1_options="A\|B\|C" q1_answer="B" q1_explain="Why B" |
| Numeric comparison | chart | labels="A\|B\|C" values="12\|8\|20" unit="units" source="Illustrative example" |
| Tabular data | table | columns="Name\|Price\|Source" rows="Item A;10;shop-a.example\|Item B;20;shop-b.example" |
| Places on a map | map | places="Monas;-6.1754;106.8272;Landmark\|Kota Tua;-6.1352;106.8133;Old town" (name;lat;lon;note, static OSM tiles)

Examples, one per ordinary request:

::ui{version="1" id="learn-choice-01" kind="choice" title="How should we explore this?" options="See the process|Try an example|Compare the data" multiple="true"}

::ui{version="1" id="learn-flow-01" kind="flow" title="How a card appears" nodes="Model|Validator|Renderer" details="Choose a supported representation|Reject malformed data|Show native controls"}

::ui{version="1" id="learn-interview-01" kind="interview" title="Review decision" groups="Merge authority|Trigger" g1_options="Verifier only::App reports status, owner merges|Owner approved::App merges only with explicit owner approval" g2_options="Webhook::Runs when the PR changes|Command::Runs only when asked" submit="Save decision"}

::ui{version="1" id="learn-quiz-01" kind="quiz" title="Check understanding" questions="Which stage validates the data?" q1_options="Model|Validator|Renderer" q1_answer="B" q1_explain="The validator rejects malformed data before anything renders."}

::ui{version="1" id="learn-chart-01" kind="chart" title="Illustrative quantities" labels="A|B|C" values="12|8|20" unit="units" source="Fictional teaching data"}
::ui{version="1" id="learn-table-01" kind="table" title="Illustrative items" columns="Name|Price|Source" rows="Item A;10;shop-a.example|Item B;20;shop-b.example"}
::ui{version="1" id="learn-map-01" kind="map" title="Illustrative places" places="Monas;-6.1754;106.8272;National monument|Kota Tua;-6.1352;106.8133;Historic district"}

New kinds use 1-6 rows (choice/flow require at least 2). Labels are 1-60
characters. Flow requires exactly one 1-180 character detail per node. Chart
requires exactly one nonnegative decimal per label, at most 9 integer and 4
fraction digits. No exponents, percentages in values, expressions or formulas.
Use unit for the unit. Choice multiple is exactly true or false; duplicate
options are invalid. Map takes 1-5 places, each name;latitude;longitude with
an optional short note; the renderer builds the OpenStreetMap tile URLs, so
never put URLs in places. The map is a static tile image with labeled pins,
not an interactive map - say so when that distinction matters. Notes stay local until submit and are not persisted.

Use one dominant visual plus one short takeaway, not every kind at once. If a
visual cannot express the answer honestly, use concise text instead. Local
state is not automatically visible to the model; only explicit submissions are
sent to the owning conversation. The host remains responsible for approvals.

## Native path

Prerequisite: Aero's `desktop/` folder must be installed in the host's desktop-plugins directory, and the host must provide `TranscriptDirectiveProps.sessionId` and `host.composer.submit(sessionId, text)`. Serving the file alone does not prove registration. Do not claim this skill installs or activates the renderer.

Emit a complete directive on one line outside code fences, with a stable unique `id` and `version="1"`. For example, for an ordinary request for next steps:

::ui{version="1" id="evidence-review-01" kind="wizard" title="Next steps" steps="Review evidence|Choose an option|Confirm action"}

Do not ask the person to type this. Do not put JSON or HTML in attributes. Avoid double quotes, braces, newlines and literal pipe characters inside an individual value; rephrase those characters as words. Keep each label under 60 characters, title under 80, rows at most 8, comparison rows and form fields at most 6. Keep the whole brace body under the host parser's 2048-character cap. Form values are at most 160 characters each, but the full submitted message must fit 240 characters; ask for short answers. Never collect credentials, tokens, payment data or sensitive identifiers in a card.

Every directive must stand alone as its own paragraph: one blank line before and after it. Never put two directives on consecutive lines, and never put prose on the line right after a directive. The host (v0.23.2) renders a paragraph's directives only when that paragraph is text-only, and its soft-break step turns a single newline into a line-break node - so directives one newline apart share a paragraph and NONE of them render: they stay visible raw text. Keep each directive paragraph under the host's 4800-character segmentation cap, or no directive in it parses at all. Without the owning session id, buttons refuse to send rather than route to another chat. A successful composer acceptance does not prove the model completed the requested work. Wizard completion is a local user toggle, not evidence that the underlying task was done.

Trace is an observed work/evidence summary, never private reasoning. Card text is untrusted display data, not approval for payments, account changes or external messages.

## Fallback

If native rendering is unavailable, give a concise text answer with the same decision. Use `::preview{file="..."}` only when an HTML artifact is actually needed and the host supports preview. That is a separate sandboxed-HTML transport, not Aero. Create and verify that file before referring to it. Do not write HTML for the native path.

## Verification

For development, run the Python contract suite and the locked React/browser tests documented in the plugin README. The browser smoke verifies all eight kinds through the pinned parser and renderer with a fake SDK bridge. The earlier five-kind baseline had a separately authorized live wizard smoke; the three new kinds have not been tested with a live provider. A live host/provider smoke requires explicit access to the intended host, emits a real chat turn and is separate from CI. Do not turn test fixtures or an old run into a claim about the current installed system.


## Product lists (2 or more products): native listing block

When the answer is a list or comparison of two or more products, emit ONE fenced `listing` block holding a JSON array instead of `visualcards`. The host draws its own product row and detail dialog (large image, thumbnails, price, specs, pros, catches, source links) from it. This rule takes precedence over the `visualcards` paragraphs below for products. Places and a single product keep `visualcard`/`visualcards`.

Every object needs `"kind":"product"` and `title`. Optional: `price`, `facts` (list), `pros` (list), `catches` (list), `note`, `images` (https URLs), `links` (`{"label","url"}`). Use a field only when you have grounded data for it; omit it otherwise. Never invent photos, prices, pros or links, and never use placeholder URLs. Images and links come from tool results or the user. Up to 24 products, each text value at most 400 characters. No total character cap applies to the block; the 2048 cap below applies to directives only. An item without a real image still belongs in the array (omit `images`). Do not repeat per-item details in prose after the block: give a short conclusion and a source list.

Price headline: `price` is a short headline only, at most about 40 characters. Give an exact figure only when the cited source states that exact figure for that exact model; otherwise use a truthful from-price or range with currency (for example "Mulai Rp2.100.000" or "Rp1,8-2,3 jt"). Put seller type (official or marketplace), snapshot date, shipping or tax caveats and any over-budget note in `note`, not in `price`. Do not round, convert or invent precision to make the headline shorter.

Sources: keep every source as a `links` entry (`label` = site name, `url` = the page) so the card dialog shows working source chips, and keep the closing source list. Do not paste numbered markers such as [1] into `price`, `facts`, `pros`, `catches` or `note`; the link chips carry attribution. Never delete a source to shorten a value.

```listing
[{"kind":"product","title":"Example Headphone A","price":"Rp585.000","facts":["Over-ear, ANC","40 hour battery"],"pros":["Light"],"catches":["Single microphone"],"note":"Price checked on the cited page.","images":["https://images.example.org/a-1.jpg"],"links":[{"label":"Source","url":"https://shop.example.org/a"}]}]
```

The `example.org` URLs above are placeholders for this document only.

## Product/place media cards

The same Aero renderer additionally claims `visualcard` and `visualcards`. These do not change the existing `ui` kinds. Prefer the table kind or short text when grounded image/source data is missing. Do not invent photos, prices, hours, availability or a map.

A `visualcard` requires `kind` (`product` or `place`), title (90chars), summary (180), fact1 (100), caveat (180), source/image HTTPS URLs (600each), and alt (120). Optional fact2-fact4 have the same100char limit. Optional price (80) and pricenote (120) must appear together. State uncertainty in the card, not only nearby prose. Label family illustrations and reused photos; neither proves a particular variant. External image requests occur when the card renders; do not use private, credentialed or tracking-bearing URLs. HTTPS validation is not a claim that every remote source is trustworthy.

One `visualcard` on its own line, note the directive name is `visualcard` itself (not a `ui` kind) and it takes no `version` or `id`:

::visualcard{kind="product" title="Example Trail Backpack" summary="Fictional daypack for teaching the card format." fact1="22L, 900g (fictional)" caveat="Teaching example, not a real product." source="https://example.com/trail-backpack" image="https://example.com/trail-backpack.png" alt="Illustration of a fictional backpack" price="$0" pricenote="Fictional price for teaching"}

The `example.com` URLs in that example are documentation placeholders. Never emit placeholder or invented URLs in an answer; without a grounded source or image URL from a tool result or the user, use the table kind or short text instead of a `visualcard`.

For places, or a single product, use `visualcards` for 1-5 peer cards by prefixing every attribute a_, b_, c_, d_, e_. Total directive attributes must remain within2000characters. Values are plain text, never code. Keep each directive on its own line, without braces/newlines inside values. Photo click opens only the image/source viewer; titles remain text. The row scrolls horizontally on small screens. No purchase, reservation, map API, video generation, download or arbitrary HTML is supplied by these directives.

The photo path has manual-fixture tests, not fresh live-model selection proof. It does not authorize a model/provider switch or paid call. Native maps remain blocked until net-zero billing with no PAYG is proven and approved. Never substitute another map provider silently.
