# Agent Template

A template repository that provides guidelines, best practices, and conventions for Python projects working with AI assistants (like GitHub Copilot).

## What is this?

This template defines:

- **Agent Instructions** - Guidelines for AI assistants on how to work with this codebase
- **Coding Standards** - Comprehensive style guide, linting rules, and code quality conventions
- **Project Structure** - Recommended layout with `src/`, `tests/`, `docs/`, and configuration files
- **Dependency Management** - Best practices using `uv` for reproducible Python environments

## Key Features

- **Clear conventions** for naming, formatting, and code organization
- **Automated tooling setup** with ruff, pyright, and other linters
- **AI-friendly documentation** to help agents and developers understand code expectations
- **Testing guidelines** for unit tests, integration tests, and edge cases

## Quick Start

1. Use this template as a starting point for new Python projects
2. Review `AGENTS.md` for AI agent instructions  
3. Check `docs/coding-instructions.md` for detailed coding rules
4. Set up your project with `pyproject.toml` at the root
5. Use `uv` for dependency management

## Files Overview

- `AGENTS.md` - Instructions and rules for AI assistants
- `docs/coding-instructions.md` - Complete coding style guide
- `pyproject.toml` - Project metadata and dependencies (should be in your project)
- `src/<package>/` - Your package source code
- `tests/` - Test suite

## Computer Vision Framework

This template includes documentation for building a Computer Vision machine learning framework.

### Subagents (`docs/subagents/`)

| File | Role |
|------|------|
| `cv-data-agent.md` | Dataset loading, preprocessing, and augmentation pipelines |
| `cv-training-agent.md` | Model training, optimization, and checkpoint management |
| `cv-evaluation-agent.md` | Metrics computation, visualization, and model analysis |

### Skills (`docs/skills/`)

| File | Task |
|------|------|
| `image-classification.md` | Single-label and multi-label image classification |
| `object-detection.md` | Bounding box detection (YOLO, DETR, Faster R-CNN) |
| `image-segmentation.md` | Semantic and instance segmentation |
| `data-augmentation.md` | Augmentation pipelines using albumentations |

## Resources

- [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html) - Referenced for style conventions
- [Ruff](https://docs.astral.sh/ruff/) - Fast Python linter and formatter
- [Pyright](https://github.com/microsoft/pyright) - Static type checker
- [UV](https://github.com/astral-sh/uv) - Python package manager
