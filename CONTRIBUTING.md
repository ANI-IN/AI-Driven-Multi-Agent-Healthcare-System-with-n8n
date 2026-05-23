# Contributing to HealthSense AI

Thank you for your interest in contributing to this project. This guide will
help you get started.

## Table of Contents

- [Getting Started](#getting-started)
- [How to Contribute](#how-to-contribute)
- [Workflow Guidelines](#workflow-guidelines)
- [Branch and PR Conventions](#branch-and-pr-conventions)
- [What Is In Scope](#what-is-in-scope)
- [What Is Out of Scope](#what-is-out-of-scope)
- [Reporting Bugs](#reporting-bugs)
- [Suggesting Features](#suggesting-features)

## Getting Started

### Prerequisites

- [n8n](https://n8n.io/) version 1.70 or later (Cloud or self-hosted)
- An OpenAI API key with access to GPT-4o
- A Microsoft Azure AD app registration with:
  - Microsoft Graph permissions for reading Excel files on SharePoint
  - Mail.Send permission for Outlook email
- An Airtable personal access token

### Local Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/ANI-IN/AI-Driven-Multi-Agent-Healthcare-System-with-n8n.git
   cd AI-Driven-Multi-Agent-Healthcare-System-with-n8n
   ```

2. Copy the environment template:
   ```bash
   cp .env.example .env
   ```

3. Fill in your credentials in `.env`.

4. Start n8n (self-hosted with Docker):
   ```bash
   docker run -it --rm \
     --name n8n \
     -p 5678:5678 \
     --env-file .env \
     -v n8n_data:/home/node/.n8n \
     docker.n8n.io/n8nio/n8n
   ```

5. Import each workflow JSON file via the n8n web UI
   (Settings > Import from File).

6. Configure credentials in n8n for OpenAI, Microsoft Excel, Microsoft
   Outlook, and Airtable.

7. Activate the Coordinator Agent workflow.

## How to Contribute

1. Fork the repository.
2. Create a feature branch from `main`:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. Make your changes.
4. Test your workflow changes by importing into a local n8n instance and
   running a manual execution.
5. Commit with a clear message following the Conventional Commits format:
   ```bash
   git commit -m "feat: add appointment cancellation workflow"
   ```
6. Push your branch and open a Pull Request against `main`.

## Workflow Guidelines

When modifying or adding n8n workflows:

- **Export cleanly**: Use n8n's "Download" feature to export the workflow.
  Make sure no real credentials are embedded in the JSON.
- **Name nodes clearly**: Every node should have a descriptive name that
  explains its purpose (not "Node 1" or "HTTP Request").
- **Add error handling**: Use the Error Trigger or "Continue on Error" setting
  on nodes that call external services.
- **Document system prompts**: If you change an AI Agent's system prompt,
  explain the change in your PR description.
- **Test with sample data**: Include a test query in the `pinData` field or
  describe the test case in your PR.

## Branch and PR Conventions

- Branch names: `feature/short-description`, `fix/short-description`,
  `docs/short-description`
- Commit messages: Use Conventional Commits (`feat:`, `fix:`, `docs:`,
  `chore:`, `refactor:`)
- PRs should include a summary of what changed, why, and how to test it.

## What Is In Scope

- New agent workflows that extend HealthSense AI's capabilities
- Improvements to existing agent system prompts
- Additional datasets (with proper licensing and no PII)
- Documentation improvements
- Bug fixes in workflow logic
- CI/CD and tooling improvements

## What Is Out of Scope

- Changes to n8n core (report those to the n8n project)
- Features that require real patient data or PII
- Integrations with paid services that have no free tier
- Non-English translations at this time

## Reporting Bugs

Use the [Bug Report template](https://github.com/ANI-IN/AI-Driven-Multi-Agent-Healthcare-System-with-n8n/issues/new?template=bug_report.md)
and include:

- n8n version
- Which workflow and node the bug occurs in
- Steps to reproduce
- Expected versus actual behavior
- Relevant execution logs (redact any credentials)

## Suggesting Features

Use the [Feature Request template](https://github.com/ANI-IN/AI-Driven-Multi-Agent-Healthcare-System-with-n8n/issues/new?template=feature_request.md)
and describe:

- The problem you are trying to solve
- Your proposed solution
- Any alternatives you considered
