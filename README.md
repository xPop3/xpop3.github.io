# xPOPE Projects

My personal project site, live at **[xpop3.github.io](https://xpop3.github.io)**.

It's where I document what I'm working on in cybersecurity, mainly [HackTheBox](https://www.hackthebox.com) machine writeups and walkthroughs.

> **Note:** HTB writeups are published only after a machine retires, in line with HackTheBox's rules on active content.

## What's here

- **HTB writeups**: step-by-step walkthroughs of retired machines, from recon and enumeration to foothold, privilege escalation, and lessons learned.
- **Projects**: other security and tech work as it comes together.

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

## Adding a writeup

Create a file in `_posts/` named `YYYY-MM-DD-machine-name.md`:

```markdown
---
title: "HTB: MachineName"
description: Short summary of the box
tags: [htb, linux, easy]
---

Writeup content goes here.
```

Push to `master`, and the site rebuilds and goes live within a couple of minutes.

## Credits

The theme is based on [Klisé](https://github.com/piharpi/jekyll-klise) by [piharpi](https://github.com/piharpi), which is MIT licensed.

## License

The site code is available under the [MIT License](LICENSE). Writeup content © xPop3.
