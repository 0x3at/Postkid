# Repository Context for AI Models

This document provides essential context about the `/home/dunder/repos/Postkid/` repository for AI models.

## 1. Repository Purpose

This repository is a personal full-stack portfolio project under active development. Its primary purpose is to showcase the developer's expertise across multiple technology stacks by implementing the *exact same* application, called the **Postkid API Playground**, in each stack. This demonstrates feature idempotency and architectural adaptability.

## 2. The Postkid API Playground Application

- **Core Concept**: The Postkid API Playground is a self-hosted, multi-tenant platform for defining, testing, and monitoring API endpoints via a web interface.
- **Feature Idempotency**: A critical aspect is that every implementation of the Postkid application must have identical features and behavior, regardless of the technology stack used.
- **Unified API Test Suite**: To ensure this idempotency, all stack implementations are validated against a single, unified API test suite that defines the expected API contract and behavior.

## 3. Application Requirements and Details

- Detailed requirements, core features, shared infrastructure, and technical specifications for the Postkid API Playground application are documented in: `/home/dunder/repos/Postkid/docs/YellowPaper.md`

## 4. Current Development State

The project is currently in a critical preplanning phase, transitioning into active development. The focus is on:

- **Documentation Finalization**: Trimming, revising, and reviewing existing documentation to create a clear and effective development document base.
- **Optimization**: Ensuring development plans and architectural descriptions are optimized.
- **Synchronization**: Critically ensuring all documentation is in sync. Downstream documents (e.g., stack-specific architecture descriptions) must accurately reflect and abide by upstream descriptions (e.g., the general stack definitions or the Yellow Paper).

**Example Synchronization Rule**: The specific architecture document for the Django stack (`/home/dunder/repos/Postkid/docs/architecture/django.md`) must always implement and abide by the general description and requirements for the Django stack found in the primary stacks document (`/home/dunder/repos/Postkid/docs/Stacks.md`).

## 5. Licensing Information

Note that the repository includes a custom license (`/home/dunder/repos/Postkid/LICENSE`) which explicitly restricts usage by LLMs.

## 6. Key Documentation Files for Context

For comprehensive understanding, refer to the following key files:

- `/home/dunder/repos/Postkid/docs/YellowPaper.md`: Core application requirements and technical specification.
- `/home/dunder/repos/Postkid/docs/Stacks.md`: Overview and definition of each technology stack implementation.
- `/home/dunder/repos/Postkid/docs/architecture/`: Directory containing architecture details for specific stacks (e.g., `/home/dunder/repos/Postkid/docs/architecture/django.md`).
- `/home/dunder/repos/Postkid/LICENSE`: Licensing terms, including restrictions for LLMs.

This context should help AI models understand the structure, purpose, and current state of the Postkid repository.