# Contributing to Awesome DevOps Skills

Thank you for your interest in contributing! This project thrives on community input.

## How to Contribute

### Adding a New Skill

1. Fork the repository
2. Create a branch: `git checkout -b skill/my-new-skill`
3. Add your skill file under the appropriate category in `skills/`
4. Follow the [skill template](#skill-template) below
5. Submit a Pull Request

### Skill Template

Every skill file must include these sections (150+ lines):

```markdown
# Skill Name

> One-line description

## When to Use / 何时使用

Clear trigger conditions for when an AI agent should load this skill.

## Architecture / 架构

Diagrams or descriptions of the system architecture.

## Code Templates / 代码模板

Production-ready YAML, HCL, Dockerfile, or shell snippets.

## Best Practices / 最佳实践

Numbered list of battle-tested recommendations.

## Pitfalls / 常见陷阱

Common mistakes and how to avoid them.
```

### Improving Existing Skills

- Fix inaccuracies or outdated information
- Add missing code examples
- Improve translations
- Add links to official documentation

### Reporting Issues

- Use the issue templates provided
- Include reproduction steps for bugs
- Suggest improvements with clear rationale

## Code Style

- Markdown files: Follow [CommonMark](https://commonmark.org/) spec
- YAML: 2-space indentation
- HCL: Use `terraform fmt` formatting
- Dockerfiles: Follow [best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

## Review Process

1. All submissions require at least one review
2. CI must pass (link checks, linting)
3. Maintainers may request changes
4. Once approved, a maintainer will merge

## Code of Conduct

Please read our [Code of Conduct](CODE_OF_CONDUCT.md) before contributing.

## Questions?

Open a discussion in the [Issues](https://github.com/liangzhengtao/awesome-devops-skills/issues) section.
