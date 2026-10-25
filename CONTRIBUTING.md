# Contributing

Thanks for helping keep this list genuinely useful. The bar is quality, not quantity — this is a curated list, not a link dump.

## What belongs here

Resources directly about **TLS or HTTP/2 client fingerprinting**: specs and concepts (JA3, JA3S, JA4+, Akamai HTTP/2, peetprint), libraries that generate or impersonate fingerprints, tools that measure them, datasets, detection/defensive projects, reflectors, and solid articles or research.

## How to propose an entry

Open a pull request that adds your entry to the right section. Each entry must:

1. **Be real.** It must be a project, tool, document, or dataset that actually exists and does what you say. No vaporware, no "coming soon".
2. **Have a resolving link.** The URL must return HTTP 200 (or 301) when fetched. Verify it yourself before submitting, e.g.:
   ```sh
   curl -sI -L -o /dev/null -w '%{http_code}\n' <url>
   ```
   Dead links, login-walled pages, and redirects to unrelated content will be rejected.
3. **Carry an honest one-line description.** State what it is plainly — language, approach, and scope. Don't oversell. For a library, name the language. For an article, say what it covers. Marketing copy will be edited or removed.
4. **Sit in the correct section** and read naturally against the entries around it.

## Format

Follow the existing style:

```md
- [Name](https://example.com) — one honest sentence describing what it is and, for code, the language and approach.
```

- Keep entries alphabetical within a section only where the section is already alphabetised; otherwise group by relevance.
- One entry per line.
- Prefer primary sources (the project's own repo or the spec author's page) over third-party reposts.

## Removing entries

If a link dies or a project is abandoned/misrepresented, a PR that removes or updates it is just as welcome as one that adds something.

## License

By contributing you agree that your contributions are released under [CC0 1.0](LICENSE).
