# portfolio

Source of [arjunharitsa.github.io/portfolio](https://arjunharitsa.github.io/portfolio/) — personal site
and blog (English and ಕನ್ನಡ) built with [Hugo](https://gohugo.io) and the
[Polar](https://github.com/ArjunHaritsa/hugo-theme-polar) theme (git submodule).

## Local development

Requires Hugo ≥ 0.158 (the standard edition is enough; CI uses 0.170.0).

```sh
git clone --recurse-submodules https://github.com/ArjunHaritsa/portfolio.git
cd portfolio/weblogs
hugo server
```

New post: `hugo new content posts/en/my-post.md` (or `posts/kn/...`); the archetype fills in the
`language` from the folder. Set `math = true` in a post's front matter to load MathJax for it.

## Layout

| Path | Purpose |
| --- | --- |
| `weblogs/content/` | Posts, About and Experience pages |
| `weblogs/static/images/` | Profile photos |
| `weblogs/hugo.toml` | Site configuration (menu, default accent, post languages, giscus) |
| `weblogs/themes/polar/` | Theme (submodule) |

## Deployment

Pushing to `main` builds the site with GitHub Actions and publishes it to GitHub Pages
(`.github/workflows/hugo.yaml`). Build output (`weblogs/public/`) is not committed.