# blog

Personal blog built with [Hugo](https://gohugo.io/) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme, deployed to GitHub Pages via GitHub Actions.

Live at: https://sukhman31.github.io/blog/

## Local development

```bash
hugo server -D
```

Then open http://localhost:1313/blog/

## New post

```bash
hugo new content posts/my-post-title.md
```

Set `draft = false` in the front matter when ready to publish, then commit and push to `main` — the GitHub Actions workflow builds and deploys automatically.
