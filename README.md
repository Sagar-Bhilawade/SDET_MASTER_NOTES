# SDET Master Notes

## 🎯 The Goal
This repository is an evolving **Living Knowledge Base** for SDET architectural patterns and technical deep-dives. It bridges the gap between official documentation and production-grade implementation, serving as a long-term reference for building and maintaining enterprise-scale automation frameworks.

## 👨‍💻 Professional Context (Sagar B.)
The content here is battle-tested and modeled after the following architectural pillars:

1.  **Cloud-Native E2E (Playwright / TS / Kubernetes):** * Focus on dynamic environment testing, K8s namespace isolation, and reducing CI/CD feedback loops via GitHub Actions and parallel execution.
2.  **Enterprise API Validation (RestAssured / Java / SQL):** * Focus on full contract validation, Request/Response SpecBuilders, and DB-to-API data integrity verification.
3.  **Legacy Web Regeneration (Selenium / Java / POM):** * Focus on stabilizing monolithic web applications, eliminating flakiness through Fluent Waits, and Jenkins Master-Slave distributed execution.

## 📂 Repository Structure
This repository is organized by **Domain**, allowing you to focus your revision on specific interview tracks.

### 01_API_Automation
*   **01_Foundations**: Core concepts (Statelessness, Idempotency, HTTP).
*   **02_Postman**: Masters of Scripting, Newman, and Mocking.
*   **03_RestAssured_Java**: Pure Java API framework design.

### 02_UI_Automation
*   **01_Selenium_Java**: Stabilizing legacy monoliths and POM patterns.
*   **02_Playwright_TS**: Modern E2E, Fixtures, and Tracing.

### 03_Infrastructure_Cloud
*   **Docker, Kubernetes, GitHub Actions, and Jenkins.**

### 04_Database_Security
*   **SQL/NoSQL testing, JWT/OAuth, and Data Integrity.**

### 05_Design_Patterns
*   **Creational, Structural, and Behavioral patterns in Test Automation.**

### 06_Build_Tools
*   **Maven, Gradle, and Multi-module project structures.**

## 🛠️ Note Generation Protocol
We follow a **"User-First Research"** flow:
1.  **User Research:** The user reads in-depth documentation from official sources.
2.  **AI Synthesis:** The agent synthesizes these findings into high-density notes using the **AGENT_INSTRUCTIONS.md**.
3.  **Gap Analysis:** The agent proactively suggests "Industry Extras" that were missing from the official docs.
4.  **Slow & Deep:** We prioritize technical depth and edge-case coverage over speed.