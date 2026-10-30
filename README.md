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
| [signup-abuse-prevention](skills/signup-abuse-prevention/SKILL.md) | Security | Reduces bot abuse on signup and waitlist forms of small SaaS products: fake accounts, inflated usage statistics, drained resources, and damage to sender email reputation. Starts from a small baseline (Turnstile, honeypot, email verification, rate limits, field validation) and adds safe email sending (send gates, bounce and complaint handling, suppression) and optional risk tiers such as phone OTP, IP intelligence, and per-IP account caps. |

## Notes

- **legal-compliance** is not legal advice. Laws change often. A lawyer must review its output before launch.
- **signup-abuse-prevention** uses illustrative limits. Tune them with your own traffic. Provider prices and limits change, so check the linked sources before you rely on them.

## Contributing

Each skill lives in `skills/<name>/` with a `SKILL.md` file. The file must start with `name` and `description` in its frontmatter.

## License

[MIT](LICENSE)
