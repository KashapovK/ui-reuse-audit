# Security Policy

## Reporting a vulnerability

Please do not report security-sensitive findings in a public issue.

Use GitHub's private vulnerability reporting for this repository when available, or contact the repository owner privately through GitHub.

Include only the information needed to reproduce and assess the issue. Do not publish credentials, tokens, private repository contents, internal URLs, proprietary design artifacts, or personal data.

## Scope

This repository contains an instruction-only Agent Skill and a skills-only plugin manifest. It does not ship a runtime service, bundle an MCP server, install hooks, execute untrusted project code, or require credentials as part of normal operation.

The skill is designed to prefer local consumer evidence first. GitHub, Figma, package registries, documentation sites, and other network sources are optional evidence providers rather than runtime dependencies.

Security-relevant findings can still include:

- instructions that encourage disclosure of private source code, credentials, internal endpoints, proprietary Figma content, or personal data to external services;
- instructions that bypass the authorization boundaries of a connected repository, design tool, or package registry;
- unsafe treatment of untrusted repository content as executable instructions;
- CI or packaging supply-chain weaknesses;
- plugin metadata that unexpectedly enables apps, hooks, MCP servers, write capabilities, or other side effects outside the documented skills-only scope.

## Safe evidence handling

Use the minimum evidence required for a classification. Prefer local imports, manifests, lockfiles, workspaces, installed package exports, and local adapters before consulting external sources.

Do not copy sensitive repository or design content into public search/documentation services merely to strengthen an audit. When private connected systems are used, preserve their existing access controls and retrieve only the evidence necessary for the task.

The skill itself must not mutate source code, Figma, package releases, issue trackers, routes/APIs/actions, or other external state as part of the audit.
