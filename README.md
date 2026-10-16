# xPOPE Projects

My personal project site, live at **[xpop3.github.io](https://xpop3.github.io)**.

It's where I document what I'm building and learning in cybersecurity and tech. Each post covers one project: what it is, how I built it, and what I learned along the way.

## Built with

- [Jekyll](https://jekyllrb.com), a static site generator
- The [Klisé](https://github.com/piharpi/jekyll-klise) theme, customized
- GitHub Pages, deployed automatically by GitHub Actions on every push to `master`

## Run it locally

You need Ruby and Bundler installed.

```bash
git clone https://github.com/xPop3/xpop3.github.io.git
cd xpop3.github.io
bundle install
bundle exec jekyll serve
```

Then open `http://localhost:4000`.

## Adding a project

Create a file in `_posts/` named `YYYY-MM-DD-project-name.md`:

```markdown
---
title: "Project Name"
description: One-line summary of the project
tags: [security, tooling]
---

What the project is, how it works, and what I learned.
```

Push to `master`, and the site rebuilds and goes live within a couple of minutes.

## Credits

The theme is based on [Klisé](https://github.com/piharpi/jekyll-klise) by [piharpi](https://github.com/piharpi), which is MIT licensed.

## License

The site code is available under the [MIT License](LICENSE). Project write-ups © xPop3.
