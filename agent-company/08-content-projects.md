# 08 — Content Projects (Non-Code)

Content projects use the same system: idea → change → issue → PR → gates → merge.
The difference: the "product" is markdown, and **merge does not mean publish**.

## Do you need OpenSpec for content?

| Content item | Use OpenSpec change? | Why |
|---|---|---|
| A single script, post, or article | **No.** Use a direct issue with a brief. | Proposal → design → spec delta is too much process for a 60-second script. |
| A series (e.g., 8-part "Modern Java" video series) | **Yes.** | The series needs a shared plan, order, and rules. |
| Brand, voice, and format rules | **Yes, as living specs.** | `openspec/specs/brand-voice/`, `openspec/specs/short-video-format/`. Agents read them on every run. |

So: content **rules** live in OpenSpec specs. Single content **items** are issues with a brief.

## Content repo layout

```
content-studio/
├── CLAUDE.md                        # Audience, channels, learned rules
├── openspec/specs/
│   ├── brand-voice/spec.md          # Tone, banned words, claims policy
│   ├── short-video-format/spec.md   # Hook ≤ 3 s, 130–160 words, 1 CTA
│   └── article-format/spec.md
├── briefs/                          # Optional: long briefs
├── scripts/2026/10/<slug>.md        # Short-video scripts (editor baseline)
├── scripts/2026/10/<slug>.txt       # Teleprompter version
├── articles/<slug>.md
├── calendar.yml                     # Planned items per week
├── tests/                           # Structure checks (word count, sections, CTA)
├── .vale/                           # Vale styles: brand terms, banned words
└── evals/brand-voice.yaml           # promptfoo rubric + golden examples
```

## Brief (issue body template)

```markdown
Type: short-video-script | article | thread
Topic: Kafka consumer groups in 60 seconds
Audience: junior backend devs
Channel: TikTok, Shorts
Goal: follow + visit course.quythai.dev
Must include: 1 concrete example
Must avoid: unverifiable performance numbers
Due: Fri
Acceptance: passes all content gates; hook ≤ 3 s; 130–160 words; 1 CTA
```

Labels: `agent-ready`, `type:content`, `priority:p2`, `risk:low`.

## Content gates

| Gate | Tool | Blocks? |
|---|---|---|
| Markdown structure | markdownlint | Yes |
| Banned words, brand terms, style | Vale | Yes |
| Length, hook length, sections, 1 CTA | `tests/` script | Yes |
| Links work | link checker | Yes |
| Frontmatter schema (title, channel, status) | JSON schema check | Yes |
| Facts with a number have a source | Script: every number has a `[^src]` footnote | Yes |
| Brand voice | promptfoo `llm-rubric` vs. golden examples | No — score in PR |
| Hook strength | promptfoo `llm-rubric` | No — score in PR |

## Content flow per week

| Day | What happens |
|---|---|
| Monday (content day) | `editor` proposes the week's items from `calendar.yml` and the initiative. You approve in batch (10–20 min). Approved items become issues. |
| Monday–Thursday | Dispatcher runs content issues. Editor opens one PR per item. Gates run. `risk:low` items merge. |
| You, when you record | Open the merged script. Record. Your edits to the final version go back as a commit. |
| Sunday | `learning` agent compares the agent draft vs. your final version. It proposes voice rules. |

## Publishing stays human

The agent output is the **merged file**.
You publish (upload video, post article) yourself.
Reasons:
- Publishing is public and hard to undo.
- Platform accounts have no safe automation path that keeps you in control.

Later, you can add a "ready to publish" step that prepares assets (title, description, tags, thumbnail text) in the PR. The final click stays yours.

## Video tasks that already have skills

You already have skills for these. Connect them as follow-up steps in the content flow:
- `short-video-script` → produces the `.md` editor baseline and the `.txt` teleprompter file.
- `trim-silence-video`, `hardsub-vietnamese-video`, `reframe-video-16x9-9x16` → post-recording steps. These run on your laptop because the video files are local and large. Keep them as local commands, not CI jobs.
