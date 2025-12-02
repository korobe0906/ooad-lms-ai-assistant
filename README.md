# OOAD LMS AI Assistant

A reference backend for an Udemy-like LMS, designed with **Object-Oriented Analysis and Design (OOAD)** and integrating:

- An **NLP Teaching Assistant Chatbot** using **RAG + LLM**.
- A **Two-Tower Recommendation System** for personalized course suggestions.

The goal is to demonstrate how to apply **clean OOAD practices** to a real AI/NLP problem, instead of just “getting the model to run”.

---

## 1. Project Goals

### 1.1. Main Objectives

- Design and implement a backend LMS with:
  - **LMS core**: user, course, lesson, enrollment.
  - **Chatbot subsystem**: Course Q&A, General Q&A, course recommendations via chat, study plan generation.
  - **Recommendation subsystem**: home-page recommendations and similar-course suggestions.

- Showcase a full OOAD workflow:
  - Requirements analysis, actors, and use cases.
  - **Domain model** and UML diagrams.
  - Architectural and class design.
  - Mapping the design to actual Python code (FastAPI).

### 1.2. Problem Statement

For learners:

- It’s hard to build a **learning path**: what to start with, what to learn next, how long it will take.
- They often have questions during the learning process but **instructors are not always available**.
- Traditional recommendation is usually “top-selling / highest-rated”, not deeply **personalized**.

For many AI projects:

- Model integration is often done in an ad-hoc way (e.g., “just call the LLM inside the route”), which:
  - Mixes domain logic and AI logic.
  - Makes it hard to switch LLM providers, vector stores, or recommendation models.

This project aims to solve that by designing a **layered, modular architecture** that clearly separates:

> LMS domain logic, NLP chatbot logic, and recommendation logic  
> so we can swap **LLM, vector store, or recommender model** without breaking the whole system.

---

## 2. Architecture & Design

### 2.1. High-Level Architecture

The system is organized into three main subsystems:

1. **LMS Core Subsystem**
   - Manages `User`, `Student`, `Instructor`, `Course`, `Lesson`, and `Enrollment`.
   - Exposes APIs to:
     - List courses.
     - Enroll in courses.
     - Retrieve course structure for study-plan generation.

2. **Chatbot Subsystem**
   - Exposes a public endpoint for the chat widget.
   - Responsibilities:
     - Intent detection (Course Q&A, General Q&A, Recommend, Study Plan).
     - Session and context management (current course, last intent).
     - RAG retrieval from the vector store.
     - LLM calls for answer generation.
     - Delegating to `RecommendationService` and `StudyPlanService` when needed.

3. **Recommendation Subsystem**
   - Encapsulates the **Two-Tower model** logic:
     - Home-page recommendations (“Recommended for you”).
     - Similar-course recommendations on the course detail page.
   - Provides public APIs so other services can reuse the recommender.

The implementation follows a **Layered Architecture**:

- **Presentation / API layer**
  - FastAPI routers: `lms.py`, `chat.py`, `recommendations.py`.

- **Application / Domain layer**
  - Services: `LMSService`, `ChatService`, `RecommendationService`, `StudyPlanService`.
  - Domain models: `User`, `Course`, `Lesson`, `Enrollment`, `ChatSession`, `ChatMessage`, `Recommendation`.

- **Infrastructure layer**
  - `LLMClient` (adapter for concrete LLMs, e.g., OpenAI).
  - `VectorStore` (adapter for FAISS/Chroma/Pinecone).
  - `TwoTowerModel` (wrapper around the actual recommendation model).
  - Storage/repositories (demo uses in-memory implementations).

### 2.2. Core Design Patterns

- **Facade**
  - `ChatService` acts as a facade for the entire NLP pipeline: NLU → Context → Retrieval → LLM → Recommendation / Study Plan.

- **Adapter**
  - `LLMClient` abstracts the LLM provider.
  - `VectorStore` abstracts vector database access.
  - This allows swapping providers (OpenAI → local model, FAISS → Pinecone, etc.) without touching application logic.

- **Strategy / Command**
  - Each intent is handled by a different strategy/command:
    - Course Q&A.
    - General Q&A.
    - Course recommendation.
    - Study plan generation.

---

## 3. Public APIs

This backend is designed as a **service** that other systems (web frontends, mobile apps, external services) can call via HTTP.
