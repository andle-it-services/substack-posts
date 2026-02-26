# Andle IT Services — Substack Posts

Source repository for all [Substack](https://substack.com/@andleitservices) posts.

Hosted via GitHub Pages at: `https://andle-it-services.github.io/substack-posts`

## Workflow

1. Write/edit post as a `.md` file in `/posts`
2. Add an entry to `posts.json`
3. Push to `main` — GitHub Pages updates automatically
4. Copy-paste the markdown into Substack editor
5. After publishing, update `substack_url` in `posts.json`

## Adding a New Post

**1. Create the file:**
```
posts/your-post-slug.md
```

**2. Add to `posts.json`:**
```json
{
  "slug": "your-post-slug",
  "title": "Your Post Title",
  "description": "One sentence description.",
  "date": "2026-03-01",
  "file": "posts/your-post-slug.md",
  "substack_url": ""
}
```

**3. After publishing on Substack**, fill in `substack_url`.

## Posts

| Date | Title | Substack |
|------|-------|----------|
| 2026-02-19 | Email Defense Without Enterprise Budget | — |
| 2026-02-13 | Hardened Docker Host for Free Using Ansible | — |

## GitHub Pages Setup

In repo Settings → Pages → Source: `main` branch, `/ (root)`.
