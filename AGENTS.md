# Repository Context for AI Models: The Postkid Project

This document serves as a comprehensive guide for AI models interacting with the `/home/dunder/repos/Postkid/` repository. It provides essential context to ensure effective and compliant assistance.

## Important Note on Licensing

Please be aware that this repository includes a custom license (`/home/dunder/repos/Postkid/LICENSE`) which explicitly **restricts usage by LLMs for training or commercial purposes**. Your interaction should strictly adhere to assisting the user with development tasks *within this repository* and not involve data extraction for external use.

## 1. Repository Purpose and Core Philosophy

This repository is a personal full-stack portfolio project under active development. Its primary and unique purpose is to **demonstrate deep full-stack development expertise across a diverse range of technology stacks**. This is achieved by:

*   Implementing the *exact same* application, the **Postkid API Playground**, in multiple, distinct technology stacks (e.g., Django, Node.js, Go, .NET).
*   Showcasing **feature idempotency**: every stack implementation must deliver identical features and behavior.
*   Proving **architectural adaptability**: demonstrating the ability to design and implement a complex application consistently across varied technological ecosystems.
*   Maintaining **rapid development cycles**: each stack implementation is targeted for completion within one week.

## 2. The Postkid API Playground Application

-   **Core Concept**: The Postkid API Playground is a self-hosted, multi-tenant platform designed to empower developers in defining, testing, and monitoring API endpoints through an intuitive web interface.
-   **Consistency is Key**: The overarching goal is that the end-user experience, API contracts, and functionalities remain *identical* across all deployed stack instances.
-   **Unified API Test Suite**: To rigorously enforce this consistency and idempotency, all stack implementations are validated against a single, unified, stack-agnostic API test suite that defines the expected API contract and behavior.

## 3. Detailed Application Requirements and Specifications

-   All detailed requirements, core features, shared infrastructure definitions, security standards, performance targets, and comprehensive technical specifications for the Postkid API Playground application are meticulously documented in: `/home/dunder/repos/Postkid/docs/Specs/Yellow Paper.md`
-   **LLM Guidance**: When considering any feature implementation or architectural decision, the `YellowPaper.md` is the **single source of truth** and must be strictly adhered to.

## 4. Current Development State and Priorities

The project is currently in a critical preplanning and initial setup phase, actively transitioning into hands-on development. The immediate focus is on:

-   **Documentation Finalization**: Trimming, revising, and reviewing all existing documentation to create an extremely clear, concise, and effective development document base.
-   **Optimization**: Ensuring development plans, architectural descriptions, and technical specifications are optimized for clarity, brevity, and actionable guidance.
-   **Synchronization**: **Critical Priority**: Ensuring absolute synchronization across all documentation. Downstream documents (e.g., stack-specific architecture descriptions) must *accurately reflect and strictly abide by* upstream, general descriptions (e.g., the general stack definitions in `Stacks.md` or the overarching `YellowPaper.md`).

**Example Synchronization Rule**: The specific architecture document for the Django stack (`/home/dunder/repos/Postkid/docs/Architecture/Python Django Stack.md`) must always implement and abide by the general description and requirements for the Django stack found in the primary stacks document (`/home/dunder/repos/Postkid/docs/Tech-Stacks/Technology Stacks.md`), which in turn must adhere to the `YellowPaper.md`.

## 5. Key Documentation Files for Comprehensive Context

For a comprehensive understanding, always prioritize and refer to the following key files:

-   `/home/dunder/repos/Postkid/AGENTS.md` (This file): General repository context and LLM instructions.
-   `/home/dunder/repos/Postkid/docs/Specs/Yellow Paper.md`: **The definitive technical specification** for the Postkid application. This is paramount.
-   `/home/dunder/repos/Postkid/docs/Tech-Stacks/Technology Stacks.md`: Overview and high-level definition of each technology stack implementation.
-   `/home/dunder/repos/Postkid/docs/Architecture/`: Directory containing more granular architecture details for specific stacks (e.g., `/home/dunder/repos/Postkid/docs/Architecture/Python Django Stack.md`).
-   `/home/dunder/repos/Postkid/LICENSE`: Licensing terms, including explicit restrictions for LLMs.

**LLM Expectation**: Your assistance should always aim to reinforce the principles of idempotency, consistency, and adherence to the `YellowPaper.md`. When asked to implement or suggest changes, always cross-reference against the defined requirements.