# Postkid: API Playground 🚀

Postkid: API Playground is a multi-stack backend application designed to empower developers in defining, testing, and monitoring API endpoints through an intuitive web interface. This project demonstrates how a consistent set of features can be implemented across various modern technology stacks, serving as both a functional tool and a technical showcase.

## Table of Contents

- [About The Project](#about-the-project)
- [Core Features](#core-features)
- [Architectural Overview](#architectural-overview)
  - [High-Level Diagram](#high-level-diagram)
- [Technology Stacks](#technology-stacks)
  - [Implemented Stacks](#implemented-stacks)
  - [Stack Comparison](#stack-comparison)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Running a Stack (General Approach)](#running-a-stack-general-approach)
- [Documentation Hub](#documentation-hub)
- [Contributing](#contributing)
- [License](#license)

## About The Project

The primary objective of the Postkid: API Playground is to provide a robust, self-hostable environment for API interaction, focusing on developer productivity. This repository contains multiple implementations of the same core application, each built with a different backend technology stack and typically paired with a corresponding modern frontend framework.

This project aims to:
- Ensure consistency in usage, features, and API requirements across all implementations.
- Demonstrate full-stack development expertise using diverse technologies.
- Adhere to rapid development cycles for each stack.
- Promote containerized deployments.

For a deep dive into the project's technical specifications, motivations, and acceptance criteria, please refer to the [**Project Yellow Paper (`docs/YellowPaper.md`)**](./docs/YellowPaper.md).

## Core Features

- **Visual API Testing**: Interactively construct and execute HTTP requests.
- **Collection Management**: Group related API endpoints into reusable collections and projects.
- **Request History**: Log all request executions with comprehensive details for debugging and reference.
- **Variable Management**: Use variables within your requests for dynamic testing.
- **Authentication Support**: Configure various authentication methods for your requests.
- **Multi-Stack Implementation**: Experience the same application built with different technologies.

## Architectural Overview

The Postkid: API Playground follows a modular, multi-stack architecture. While each stack implementation is independent, they all adhere to common principles outlined in `docs/YellowPaper.md` and `docs/architecture/readme.md`. Key shared aspects include:

- **Shared API Specification**: All backends expose the same API contract, detailed in [`docs/Endpoints.md`](./docs/Endpoints.md).
- **Unified Conceptual Data Model**: A consistent data structure is targeted by all stacks, connecting to a shared PostgreSQL database. Details on the data model can be found in the [Data Model Overview section of the Architecture README](./docs/architecture/readme.md#4-data-model-overview).
- **Containerization**: Each stack (backend and frontend) is designed to be run in Docker containers.
- **Gateway Pattern**: Production deployments typically involve an API Gateway (e.g., Nginx) for routing, SSL termination, and load balancing.

### High-Level Diagram

This diagram illustrates the general system architecture applicable to all stack implementations:

```mermaid
%%{init: { 'theme': 'base', 'themeVariables': { 'primaryColor': '#7aa2f7', 'primaryTextColor': '#1a1b26', 'lineColor': '#bb9af7', 'background': '#1a1b26', 'mainBkg': '#1a1b26'}}}%%
graph TD
    A[User] --> B{Frontend SPA};
    B --> C{API Gateway / Reverse Proxy};
    C --> D[Backend API Service (Stack Specific)];
    D --> E[Shared PostgreSQL Database];
    D -.-> F[(Optional) Shared Redis Cache];
    D -- In-Process Async --> G((External API Call));
    D --> H[Logging Service/Mechanism];

    subgraph "Client Layer"
        B
    end
    subgraph "Gateway/CDN Layer"
        C
    end
    subgraph "Application Layer (Stack Specific Implementation)"
        D
        G
        H
    end
    subgraph "Data Layer (Shared)"
        E
        F
    end

    style A fill:#c9d1d9,stroke:#768390
    style B fill:#ff9e64,stroke:#ff7043
    style C fill:#73daca,stroke:#41a6b5
    style D fill:#e0af68,stroke:#d19a66
    style E fill:#9ece6a,stroke:#73b25a
    style F fill:#a5d6ff,stroke:#89bde8
    style G fill:#f7768e,stroke:#db5a6b
    style H fill:#bb9af7,stroke:#9d7cd8
```

For a comprehensive architectural overview, including shared design principles, detailed workflow diagrams, and individual stack documentation, please visit the [**Architecture Documentation Hub (`docs/architecture/readme.md`)**](./docs/architecture/readme.md).

## Technology Stacks

This project implements the Postkid: API Playground using several distinct technology stacks. The definitive list and specifications for these stacks are maintained in [`docs/Stacks.md`](./docs/Stacks.md).

### Implemented Stacks

1.  **Python/Django + React**:
    *   Backend: Django 5.0, Django REST Framework 3.14
    *   Frontend: React 18, TypeScript 5.0, Vite 4.0
    *   Detailed Architecture: [`docs/architecture/django.md`](./docs/architecture/django.md)
2.  **Java/Spring Boot + Angular**:
    *   Backend: Spring Boot 3.2, Spring Data JPA, Spring Security 6
    *   Frontend: Angular 17, TypeScript 5.0
    *   Detailed Architecture: [`docs/architecture/springboot.md`](./docs/architecture/springboot.md)
3.  **C#/.NET + Blazor WASM**:
    *   Backend: .NET 8 Web API (ASP.NET Core)
    *   Frontend: Blazor WebAssembly (.NET 8)
    *   Detailed Architecture: [`docs/architecture/dotnet.md`](./docs/architecture/dotnet.md)
4.  **Node.js/Express + Next.js**:
    *   Backend: Node.js 20, Express.js 4.18, TypeScript, Prisma 5.0
    *   Frontend: Next.js 14 (App Router), TypeScript
    *   Detailed Architecture: [`docs/architecture/next.md`](./docs/architecture/next.md)

### Stack Comparison
For a detailed side-by-side comparison of these technology stacks, including frameworks, ORMs, authentication methods, testing tools, and more, please refer to the [**Technology Stack Comparison Table in the Architecture README**](./docs/architecture/readme.md#6-technology-stack-comparison).

## Getting Started

### Prerequisites

- Git
- Docker Engine (latest stable version recommended)
- Docker Compose (typically included with Docker Desktop)

### Running a Stack (General Approach)

Each technology stack is self-contained and can be run independently using Docker. While specific commands for a particular stack might be found within its dedicated documentation or a future monorepo setup, the general approach using Docker Compose is as follows:

1.  **Clone the repository:**
    ```sh
    git clone https://github.com/your-username/api-playground.git # Replace with the actual repository URL
    cd api-playground
    ```
2.  **Navigate to a specific stack's directory or the root (if using a monorepo with top-level compose files for each stack):**
    The project structure may involve individual `docker-compose.yml` files per stack (e.g., `stacks/django-react/docker-compose.yml`) or a centralized management system if set up as a monorepo. Refer to the specific setup instructions that will accompany each stack's implementation. A conceptual example based on `docs/YellowPaper.md`:
    ```sh
    # Example: If docker-compose.yml is at the root for a specific stack configuration
    # (or navigate to the stack's directory if they are separate)
    # docker-compose -f docker-compose.django.yml up --build
    ```
3.  **Build and run the containers:**
    ```sh
    docker-compose up --build -d
    ```
    *(The `-d` flag runs containers in detached mode. Omit it to see logs directly.)*

4.  **Access the application:**
    The frontend and backend API for the selected stack will be accessible on ports defined in their respective `docker-compose.yml` files (e.g., frontend on `http://localhost:3000`, backend API on `http://localhost:8000`).

*(Note: As the project evolves, precise `docker-compose.yml` configurations and commands for each stack will be finalized and documented within their specific directories or a unified development guide.)*

## Documentation Hub

All project documentation, including technical specifications, architectural details, and API definitions, can be found in the [`/docs`](./docs/) directory.
- [**Project Yellow Paper (`docs/YellowPaper.md`)**](./docs/YellowPaper.md): Core concepts, features, requirements, and high-level architecture.
- [**Technology Stacks Definition (`docs/Stacks.md`)**](./docs/Stacks.md): Precise definition of technologies used in each stack.
- [**API Endpoints Specification (`docs/Endpoints.md`)**](./docs/Endpoints.md): Detailed API contract common to all backends.
- [**Architecture Overview & Detailed Stack Architectures (`docs/architecture/readme.md`)**](./docs/architecture/readme.md): Central hub for architectural discussions, comparisons, and links to individual stack architecture documents.

## Contributing

We welcome contributions to the Postkid: API Playground! Whether it's bug fixes, feature enhancements, or improvements to documentation, your help is appreciated.
Please read our (forthcoming) `CONTRIBUTING.md` for details on our code of conduct, development process, and how to submit pull requests.

## License

This project is distributed under the MIT License. See the `LICENSE` file (to be created) for more information.
