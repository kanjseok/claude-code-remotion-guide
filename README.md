# Claude Code Remotion Guide

> Comprehensive technical guide for programmatic video production using Remotion (React) with Claude Code as the agentic development interface.

## Why Claude Code Remotion Guide?

- **Problem**: Remotion's powerful programmatic video framework has a steep learning curve, and integrating it with AI-assisted development workflows lacks consolidated documentation.
- **Solution**: A single, in-depth guide covering the full Remotion + Claude Code workflow — from project scaffolding to cloud-scale rendering — written for senior developers and creators.
- **Scope**: Architecture, environment setup, deterministic rendering, animation systems, audio integration, composition patterns, CLAUDE.md configuration, render pipelines, cloud scaling, advanced recipes, and troubleshooting.

## Quick Start

Open either guide and follow **Section 2: Environment Setup with Claude Code** to scaffold a new Remotion project.

```bash
npx create-video@latest my-video --template-type typescript
cd my-video
npm install @remotion/media-utils @remotion/tailwind-v4 @remotion/shapes @remotion/transitions
```

### Prerequisites

- Node.js >= 18
- React + TypeScript fluency
- Familiarity with Claude Code CLI
- Remotion v4.x / v5.x

## Available Languages

| Language | File |
|----------|------|
| English | [english.md](english.md) |
| Korean | [korean.md](korean.md) |

## Assets

The [assets/](assets/) directory contains SVG diagrams referenced in the guides:

- `01-architecture-overview.svg` — Four-layer architecture diagram
- `02-animation-cheatsheet.svg` — Animation system reference
- `03-workflow-pipeline.svg` — Render pipeline workflow

## Contributing

Contributions are welcome. Please read the [Contributing Guide](CONTRIBUTING.md) and [Code of Conduct](CODE_OF_CONDUCT.md) before opening a pull request.

## License

This work is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE).

Remotion itself uses a [Business Source License (BSL 1.1)](https://remotion.dev/license) — free for individuals and teams of 3 or fewer.

## Author

**kanjseok** — creator and maintainer

## References

- [Remotion Documentation](https://remotion.dev/docs)
- [Remotion GitHub](https://github.com/remotion-dev/remotion)
- [Remotion Templates](https://remotiontemplates.dev)
