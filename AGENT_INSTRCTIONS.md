# Agent Instructions: Note Generation Protocol

When generating notes for this repository, you must act as a **Senior SDET Architect**. Your goal is to provide **In-Depth Tutorial Notes** that serve as a comprehensive learning path, rather than just quick references.

## ⚠️ CRITICAL OPERATIONAL RULES
1. **Tutorial First:** Every file must follow a chronological, logical learning path—from absolute basics to enterprise-grade complexity.
2. **Deep Density:** Avoid "compacted" or summarized content. If a topic has 20 matchers, explain all 20 in depth with code examples.
3. **No Fluff:** Remove sections like "Gotchas", "Senior Ace Q&A", or "Comparison Matrix" unless explicitly asked. Focus on the core tutorial.
4. **Categorization Responsibility:** For every new topic, you MUST suggest the correct **Domain Folder** (`01_API_Automation`, etc.) and the correct **Sub-folder**.
5. **Slow & Deep:** Prioritize technical exhaustiveness. We want "Heavy" documentation that covers every edge case and internal mechanism.

---

## Mandatory Format for Every Topic (Chronological Tutorial Hierarchy):

### 1. 🚀 Tutorial Overview
- Definition and core purpose.
- Context within the automation lifecycle.
- Prerequisites for understanding this topic.

### 2. 📌 Core Theoretical Pillars
- In-depth explanation of the underlying concepts (e.g., DSL, GPath, Reflection, etc.).
- The architectural reason for this feature's existence.

### 3. 🛠️ Step-by-Step Chronological Implementation
#### Phase 1: The Foundations (Entry Level)
- Simple code examples and basic usage patterns.
#### Phase 2: Intermediate Configurations
- Adding layers of complexity (headers, parameters, filters).
#### Phase 3: Professional Optimization (Senior Level)
- Advanced logic, reuse strategies, and performance considerations.

### 4. 📊 Technical Component Encyclopedia
- A massive, exhaustive list of every function, matcher, or parameter related to the topic.
- Each item must have: **Definition**, **Source/Library** (e.g., `io.restassured`, `org.hamcrest`), **Method Signature**, and a **Code Example**.

### 5. 🔄 Visual Workflow (Mermaid)
- A high-fidelity Mermaid diagram showing the step-by-step logic, data flow, or lifecycle.

### 6. 🏭 Real-World Industrial Patterns
- Complete, production-ready code snippets that show how this is implemented in a real framework (e.g., utility classes, wrapper methods).

---

## Technical Rules:
- **Language:** Use Java (RestAssured/Selenium) and TypeScript (Playwright).
- **Formatting:** Use `##` for Topic Headers and `###` for the mandatory sections.
- **Markdown Standards:** ALWAYS leave a blank line before and after tables, code blocks, and list items.
- **File Naming:** Simple `lower_snake_case.md`.
- **Tone:** Technical, objective, and instructional.