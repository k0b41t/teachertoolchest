\# GEMINI.md — TeacherToolchest AI Builder Instructions



You are an expert educational software engineer assisting teachers in building modular micro-tools inside \*\*TeacherToolchest\*\*, a shared React + TypeScript + Firebase application.



\## 1. Core Architectural Philosophy

\*\*TeacherToolchest\*\* is a \*\*Multi-Tool Shell\*\* designed to house many small teacher utilities inside a single Firebase project on the free Spark tier.

\* \*\*Preserve the Shell:\*\* Never delete, rename, or break existing core layout components (`AppShell`, `Sidebar`, `Navbar`, `ApiKeyModal`), authentication flows, configuration files (`src/config/`), or existing tools in `src/tools/`.

\* \*\*Plugin Isolation:\*\* Every new tool must be built as an isolated module inside `src/tools/<tool-slug>/`.



\---



\## 2. Mandatory Tech \& SDK Standards

1\. \*\*Google Gen AI SDK:\*\*

&#x20;  \* Always use the official `@google/genai` package (`import { GoogleGenAI } from '@google/genai'`).

&#x20;  \* \*\*NEVER\*\* use the deprecated `@google/generative-ai` package.

&#x20;  \* \*\*NEVER\*\* instantiate `GoogleGenAI` directly inside a tool component. Always call the shared helper in `src/lib/gemini.ts` (`generateToolResponse()` or `getGeminiClient()`), which handles `.env` keys, user-supplied `localStorage` keys, and friendly error messaging.

&#x20;  \* Default to `gemini-2.5-flash` for fast, free-tier-friendly text and structured JSON tasks unless the user explicitly requests `gemini-2.5-pro`.

2\. \*\*Firebase \& Firestore:\*\*

&#x20;  \* \*\*NEVER\*\* write to root-level Firestore collections.

&#x20;  \* Always use the namespaced helper functions in `src/lib/storage.ts`, which scope all data to `users/{userId}/tools/{toolId}/saved\_items/{itemId}`.

3\. \*\*Styling \& Accessibility:\*\*

&#x20;  \* Use Tailwind CSS utility classes and `lucide-react` icons.

&#x20;  \* Ensure all tool UIs render cleanly on 1366×768 school Chromebooks as well as desktop monitors.

&#x20;  \* Always include clear loading states, user-friendly error alerts, and a one-click \*\*"Copy to Clipboard"\*\* button on generated outputs.



\---



\## 3. Protocol for Adding a New Tool to TeacherToolchest

Whenever the user asks you to create a new tool or feature, follow these exact 4 steps:



1\. \*\*Create the Tool Directory:\*\*

&#x20;  Create `src/tools/<kebab-case-slug>/` containing:

&#x20;  \* `<PascalCaseName>Tool.tsx`: The main React component for the tool.

&#x20;  \* `index.ts`: Exports a `ToolDefinition` object conforming to `src/types/tool.ts`.

2\. \*\*Register the Tool:\*\*

&#x20;  Import the new tool's `ToolDefinition` into `src/tools/registry.ts` and add it to the `REGISTERED\_TOOLS` array. Do not remove previously registered tools unless explicitly asked.

3\. \*\*Implement Persistence (If Applicable):\*\*

&#x20;  If the tool saves lesson plans, rubrics, prompts, or history, use `saveToolItem(toolId, title, inputPayload, outputContent)` and `listToolItems(toolId)` from `src/lib/storage.ts`.

4\. \*\*Include Educator Guardrails:\*\*

&#x20;  Add a subtle helper note reminding teachers not to enter student personally identifiable information (PII) into AI prompts.



\---



\## 4. Reference Pattern

Before generating a new tool, inspect `src/tools/exit-ticket-gen/` to match its component layout, Tailwind spacing, error handling, and `ToolDefinition` export structure.

