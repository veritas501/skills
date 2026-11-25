# README special cases

Read the section that applies. `SKILL.md` says when.

## Images and badges

- Every image has alt text that says what it shows.
- A logo or diagram must be legible on both light and dark backgrounds. When two variants exist, use a `<picture>` element with `prefers-color-scheme` sources.
- A chart or screenshot gets a one-line caption stating the scenario.
- Never reference an image that does not exist; ask for it in the reply.
- Badges fit on one row, and each links to its source and reflects something that is actually configured.
- HTML only where markdown cannot do the job, such as a centered logo.

## Editing an existing README (`patch` mode)

- Match what is there: heading style and capitalization, inline or reference-style links, line wrapping, and the author's choice about emoji.
- Do not reflow paragraphs you are not changing; it buries the real change in the diff.
- Keep heading text unchanged, since other pages may link to its anchor.
- In Chinese text, follow the file's convention for spacing between CJK and Latin characters; default to a space.

## Translations

When sibling translations exist, such as `README.zh-CN.md`, keep the language-switch links near the top. After changing one file, update the others or list them as stale in the reply.
