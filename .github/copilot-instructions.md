---
description: This file contains Copilot instructions for development.
applyTo: **
---

These instructions are intended to guide Copilot in generating code. They provide context and rules to follow when suggesting code, ensuring consistency and adherence to the project's standards.

## Guidelines

- Do not preserve backward compatibility. Remove obsolete paths instead of adding compatibility layers, fallbacks, or migrations.
- Choose the simplest implementation that fully meets the current requirements. Avoid speculative abstractions, configurations, and indirections.
- Grow the system in layers. Start from the smallest version that works end to end, and add each new capability on top of a proven foundation.
- Keep components modular and concerns clearly separated.
- Prefer established, well-maintained libraries overall complexity or improve reliability. Do not reimplement common functionality without a clear reason.
- Learn on the dependencies already in the project before writing your own implementation or adding packages. Do not assume a library lacks a capability without checking its documentation and types.
- Make architecture decisions for the long term. Do not accept stopgap that only works for now and is meant to be replaced later.
