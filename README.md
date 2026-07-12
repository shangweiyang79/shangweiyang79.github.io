# shangweiyang79.github.io

Personal site of Shangwei (Joy) Yang — portfolio and research notes.
Built with [Astro](https://astro.build), deployed to GitHub Pages on every push to `main`.

Live at **https://shangweiyang79.github.io**

## Adding a research note

1. Copy an existing file in `src/content/notes/` (any `.md` file there becomes a page).
2. Rename it — the filename becomes the URL, e.g. `my-new-note.md` → `/notes/my-new-note/`.
3. Edit the frontmatter:

   ```yaml
   ---
   title: My new note
   date: 2026-07-12
   tags: [evaluation, rag]
   summary: One sentence shown on the notes index and in link previews.
   ---
   ```

4. Write the note in Markdown below the frontmatter. Footnotes (`[^1]`) render as
   margin notes on wide screens.
5. Commit and push — GitHub Actions rebuilds and deploys automatically:

   ```sh
   git add .
   git commit -m "Add note: my new note"
   git push
   ```

## Local development

```sh
npm install
npm run dev       # http://localhost:4321
npm run build     # production build into dist/
```

## Where things live

| Path | What it is |
| --- | --- |
| `src/content/notes/` | Research notes (Markdown) — the part that updates often |
| `src/pages/` | Home, work index, project case studies, about |
| `src/components/` | Diagrams (cascade, RAG) and the sidenote component |
| `src/styles/global.css` | The whole design system: colors, type, figures, sidenotes |
| `.github/workflows/deploy.yml` | Build + deploy to GitHub Pages |
