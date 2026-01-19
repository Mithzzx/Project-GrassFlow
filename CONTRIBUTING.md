# Contributing to GrassFlow

This is intended to be a public project that anyone can participate in. Our goal is to create an open-source, high-performance standard for vegetation in Unity that matches or exceeds commercial alternatives.

## How You Can Help

We are currently focused on reaching **Version 1.0**. We are looking for help with:

- **Compute Shader Optimizations**: Identifying bottlenecks in `GrassCompute.compute` to close the performance gap.
- **Render Pipeline Support**: Implementing HDRP and Built-in Render Pipeline versions of the shaders.
- **Tooling**: Improving the painting and editing workflow.
- **Documentation**: Fixing typos or adding clarifications to the docs.

## Getting Started

1. Check the **[Roadmap](ROADMAP_V1.0.md)** to see high-priority tasks.
2. Search [Issues](../../issues) to see if someone is already working on it.
3. Fork the repository and create a branch for your feature or fix.

## Pull Requests

1. Please focus your PR on a single issue or feature.
2. Describe your changes clearly in the PR description.
3. If you are improving performance, please include benchmark numbers (e.g., "Improved frame time from 6.2ms to 4.5ms on M2").

## Code Style

- **C#**: Follow standard Unity C# conventions.
- **Shaders**: Keep HLSL code readable and comment complex math operations.
- **Structure**: Keep `GrassRenderer.cs` as organized as possible; consider splitting logic if it grows too large.
