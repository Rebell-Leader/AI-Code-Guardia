AI Code Guardian: Technical Plan

This document outlines the architecture, technology stack, and main components for implementing the AI Code Guardian. The system is designed to be a robust, extensible pair programming assistant that can be scaled from a simple hackathon MVP to a full production service.
1. Architecture Overview

The system is a multi-agent architecture with a clear separation of concerns. Communication is centralized through a custom Model Context Protocol (MCP) server, which acts as the control plane for the entire system.

graph TD
    subgraph Human Operator
        A[User Prompt/Task]
        B[Human-in-the-Loop UI]
    end

    subgraph The System
        subgraph Code Guardian - [Backend Service]
            G[Guardian Agent LLM]
        end
        subgraph fastMCP 2.0 Server - [Middleware]
            C[API Endpoints]
            D[Logging & State Management]
            E[Human-in-the-Loop Orchestrator]
        end
        subgraph Developer Agent - [Frontend/IDE]
            F[Coding Agent LLM]
        end
    end

    A --> F;
    F -- Thought/Action/Edit --> C;
    C -- Real-time Check --> G;
    G -- Correction/Critique --> C;
    C -- Interrupt/Guidance --> F;
    C -- Request Validation --> B;
    B -- Approve/Reject --> C;
    C -- Log Events --> D;

2. Technology Stack

    Backend & Agents: Python will be the primary language for all backend services.

    LLM Inference: The Cerebras inference provider will be used for both the Coding Agent and the Guardian Agent. The Cerebras Python SDK or a wrapper like LangChain will be used for API calls.

    Model Context Protocol: The fastMCP 2.0 framework will be used to build the MCP server. It's a high-level, Pythonic framework that simplifies tool and resource exposure to LLMs.

    Frontend & Visualization: A simple web-based UI for the human-in-the-loop validation and a basic timeline-style visualization of the agent's progress. This can be built with plain HTML/CSS/JavaScript for the MVP, with a roadmap to React or Vue for a production-grade system.

    Database: A simple in-memory database like SQLite for the MVP, or a more robust solution like PostgreSQL or Firestore for a production system, to store the agent's logs, actions, and interventions.

3. Main Components
3.1. The fastMCP 2.0 Server

This component is the middleware that connects the Coding Agent to the Guardian Agent. It is the most critical part of the system's architecture.

    API Endpoints:

        /v2/context/: Endpoint to receive the Coding Agent's real-time actions and "thoughts."

        /v2/context/validation/: Endpoint for the Guardian Agent to send its critique or corrected plan.

        /v2/ui/: Endpoint to serve the human-in-the-loop dashboard.

    Logic:

        Interceptor: The server will receive every action from the Coding Agent and immediately forward it to the Guardian Agent for a "second opinion."

        Orchestrator: It manages the state of the session. If the Guardian Agent returns a misaligned flag, the server will interrupt the Coding Agent's execution and inject the Guardian's feedback as a new prompt.

        Human-in-the-Loop: It pauses the process and sends a request to the UI for human validation when a critical issue is detected.

3.2. The Guardian Agent

This is the brain of the operation, implemented as a Python service.

    Logic:

        Proactive Planning: On receiving the initial task, it calls the Cerebras API with a system prompt like: "You are a senior developer. Given this task, what is the safest and most efficient plan for a junior developer to follow?" This plan is then sent back to the Coding Agent as its initial instruction.

        Real-time Critique: For each subsequent action, it calls the Cerebras API again. The prompt will be structured to compare the Coding Agent's action against the initial plan and general best practices.

    Technology: Python script using the Cerebras SDK. The logic will be primarily contained within the prompt engineering of the LLM calls.

3.3. The Dashboard (UI)

This is the interface for the human operator.

    Logic:

        A simple web page served by the fastMCP 2.0 server.

        It will display a feed of the Coding Agent's actions and the Guardian Agent's interventions in a timeline format.

        It will have a modal pop-up for the human-in-the-loop validation, showing the side-by-side comparison of the Coding Agent's proposed code and the Guardian Agent's critique.

    Technology: Plain HTML/CSS/JavaScript for a fast, minimalist MVP.
