# Skills

Agent skills for building and launching products. They work with Claude Code, Codex, Cursor and other agents that the [`skills`](https://github.com/vercel-labs/skills) CLI supports.

## Install

Install all skills:

```bash
npx skills add jesubohr/skills
```

Install one skill:

```bash
npx skills add jesubohr/skills --skill legal-compliance
```

## Skills

| Skill | Category | What it does |
| --- | --- | --- |
| [legal-compliance](skills/legal-compliance/SKILL.md) | Operations | Audits a project and writes a reviewable draft of Terms, Privacy, Cookies and Refund policies, sign-up consent checkboxes, auditable consent records, data minimization and media rights. Covers Latin America and the United States. No code changes until a developer approves the draft. |

## Notes

- **legal-compliance** is not legal advice. Laws change often. A lawyer must review its output before launch.

## Contributing

Each skill lives in `skills/<name>/` with a `SKILL.md` file. The file must start with `name` and `description` in its frontmatter.

## License

[MIT](LICENSE)
