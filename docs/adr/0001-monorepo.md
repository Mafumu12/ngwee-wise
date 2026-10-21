# 0001 — Monorepo with self-contained modules

Status: accepted · Date: 2026-10-21

## Context
Hackathon rules require one public repo with everything needed to run. After the
hackathon the work should split into three portfolio repos.

## Decision
One repo: services/wallet, services/agent, services/publishers, apps/console,
infra/, docs/adr/. Each module has its own README and tests and talks to others
only over HTTP. MIT licence.

## Consequences
Cross-module changes stay fast during the build. Clean HTTP boundaries allow a
later `git subtree split` per module with history intact.
Rejected: three repos now (slows cross-cutting changes for 24 days).
