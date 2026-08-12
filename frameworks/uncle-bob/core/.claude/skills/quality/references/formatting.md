# Formatting Reference

## Universal Rules

- Formatting is automated - run the formatter, never hand-format
- One formatter per language, configured once, enforced in CI
- Never debate style in review - point at the formatter config and move on

## TypeScript / JavaScript

Formatter: `biome format` (or Prettier - pick one per repo, never both)

- Indent: 2 spaces
- Quotes: double
- Semicolons: always
- Trailing commas: on multiline
- Max line width: 100

```bash
pnpm exec biome format --write .
# or: pnpm exec prettier --write .
```

## CSS

- One selector per line, one declaration per line
- Declaration order: layout → box model → typography → visual
- Design tokens grouped at the top of the file

## Markdown

- Fenced code blocks always declare a language
- One blank line between sections - never two or more

## EditorConfig

Commit an `.editorconfig` so every editor agrees before the formatter runs:

```ini
root = true

[*]
charset = utf-8
indent_style = space
indent_size = 2
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
```
