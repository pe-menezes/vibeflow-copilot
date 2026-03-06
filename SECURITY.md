# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in Vibeflow, please report it responsibly.

**Do not open a public issue.**

Instead, email **pedro@vibeflow.dev** with:
- A description of the vulnerability
- Steps to reproduce
- Potential impact

We will acknowledge your report within 48 hours and work on a fix.

## Scope

Vibeflow is a methodology tool that generates Markdown files and runs shell commands via the CLI installer. The main security-relevant areas are:

- **CLI (`setup-vibeflow`)** — Downloads files from GitHub and writes them to the local filesystem
- **Prompts / Commands** — Markdown files consumed by AI agents; no code execution themselves

## Supported Versions

We only support the latest version. Please ensure you're using the most recent release before reporting.
