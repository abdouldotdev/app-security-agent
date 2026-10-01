# App Security Agent

![App Security Agent icon](assets/logo.png)

An agent skill for reviewing Flutter applications against [OWASP MASVS v2.1.0](https://mas.owasp.org/MASVS/). It covers 24 controls across eight groups, from secure storage and authentication to resilience and privacy. Findings are ranked by severity and include a file location, rationale, and an actionable fix.

## Use it

Copy this directory into your agent's skills folder, then invoke the skill on a Flutter project. The main instructions are in [SKILL.md](SKILL.md); the report structure lives in [assets/security-report-template.md](assets/security-report-template.md).

- **Full audit:** review the complete Flutter project.
- **Targeted check:** review changed files with `app-security-agent check <file>...`.
- **Single group:** focus on one control family with `app-security-agent group MASVS-STORAGE`.
- **Quick check:** focus on critical and high-severity findings with `app-security-agent quick`.

The skill optionally references `flutter-best-practice` and `flutter-clean-arch`; its built-in checks can still be used without them. It produces a review report, not a security certification.

This is an independent community project. It is not affiliated with or endorsed by OWASP.

## License

MIT. See [LICENSE](LICENSE).
