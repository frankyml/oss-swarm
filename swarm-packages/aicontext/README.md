# Package: aicontext

First-party external package demonstrating the swarm package model.

## What It Provides

A single behavior (`aicontext-aware`) that teaches agents how to discover and use `.aicontext/` project context files when present in a repository.

## Trust Tier

**Tier 1** — behaviors only. Cannot add modes or agents.

## Memory Isolation

All package memory writes are scoped to `packages/aicontext/` — isolated from core and agent memory.

## Installation

Reference this package in the adapter's package registry. The adapter loads `manifest.yaml`, validates the trust tier, and injects declared behaviors into agent prompts when the package's activation conditions are met.
