# Writing Guidelines

Formatting rules for posts on this blog. These are about *presentation*, not
content — never rewrite or rephrase the author's words to satisfy a rule
here. If a rule and the original wording conflict, keep the wording and skip
the rule.

## Front matter

```toml
+++
date = 'YYYY-MM-DDTHH:MM:SS+05:30'
draft = false
title = 'Post Title'
tags = ['tag-one', 'tag-two']
+++
```

- `draft = false` only when ready to publish.
- 2-4 tags, lowercase, kebab-case.

## Headings

- Break the post into `##` (h2) sections once it's longer than ~4-5
  paragraphs. Anything shorter can stay heading-free.
- Headings are structural, not decorative — one per logical section (problem,
  approach, results, limitations, closing), not one per paragraph.
- `ShowToc = true` is already set in `hugo.toml`, so adding headings
  automatically produces a jump-to table of contents. No extra work needed.
- Don't invent a heading's wording from nothing — it should describe the
  section that already exists.

## Lists

- When a list item follows a "Label - explanation" pattern, bold the label:
  `**Label** - explanation`. This is a pure markup change (wrapping existing
  words in `**`), never rephrase the label or the explanation.
- Use numbered lists for sequential or ranked items, bullet lists otherwise.

## Tables

- If a list's items are really parallel comparisons (benchmark results, before/after
  numbers, option comparisons), convert it to a table instead of a bulleted
  list — it's far more scannable. Keep every word from the original item
  exactly as written; only the container changes (list → table cell).
- Don't force a table where the items aren't actually parallel/comparable.

## Blockquotes

- Pull out at most one or two sentences per post as a `>` blockquote — pick a
  line that's a strong standalone insight, used to break up a long
  paragraph. Copy it verbatim; don't summarize or trim it.
- Don't overuse — a blockquote on every paragraph defeats the purpose.

## Images & diagrams

- Store images under `static/images/`, reference them as
  `/blog/images/filename.png` (the `/blog/` prefix matches this site's
  `baseURL` path — without it the image 404s once deployed).
- Always write a real descriptive `alt` text (what the diagram shows), not
  just a filename or "diagram".
- Diagrams should show only what the post is actually explaining — favor
  clarity over completeness. If a source diagram (e.g. from a paper) has more
  detail than the post needs, crop or simplify it rather than dumping the
  whole thing in.
- Place the image right after the text that introduces the concept it
  illustrates, not at the top or bottom of the post by default.

## Inline code

- Wrap real identifiers in backticks: function/method names, event names,
  config keys, file paths, CLI flags (e.g. `` `agent.died` ``,
  `` `AcquireLease` ``).
- Don't backtick proper nouns or product names (`AgentKeeper`, `ZooKeeper`,
  `Raft`) just because they sound technical — backticks mean "this is
  literal code/syntax," not "this is a technical word."
- If a post has no real identifiers to reference, skip this rule entirely
  rather than forcing it.

## What not to do

- Don't add a TL;DR, summary box, or closing call-to-action unless the
  author writes it themselves or explicitly approves specific wording —
  those are new sentences, not formatting.
- Don't merge, cut, or reorder paragraphs to "improve flow" — formatting
  changes structure around the words, it doesn't touch the words or their
  order.
- Don't add filler visuals (stock icons, decorative dividers) that don't
  carry information.
