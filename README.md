# Chaotic Minds Gaming Documentation

Architecture, implementation plans, security standards and operator guides for the Chaotic Minds Gaming (CMG) platform.

## Project principles

- PC-first, staged launch.
- First game integrations: ATLAS and 7 Days to Die.
- ARK: Survival Evolved prepared as the next game adapter.
- One shared Core, modular game integrations, separate website and Discord bot applications.
- Free-first infrastructure. No paid services or production deployments without explicit approval.
- Least privilege; no production changes during development.
- Changes are developed on feature branches and reviewed through pull requests.

## Repository map

See [architecture/REPOSITORY-MAP.md](architecture/REPOSITORY-MAP.md) for current repository responsibilities and dependency direction.

## ATLAS implementation status

See [plugins/ATLAS-IMPLEMENTATION-REGISTRY.md](plugins/ATLAS-IMPLEMENTATION-REGISTRY.md) for the canonical repository ownership matrix, implementation maturity, duplicate-boundary decisions, integration blockers and current consolidation snapshot.
