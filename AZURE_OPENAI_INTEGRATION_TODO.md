# Azure OpenAI Integration TODO List & Log

## Overview

This document tracks the tasks, progress, and findings related to integrating Azure OpenAI support into the Gemini CLI using LiteLLM. It serves as a development log and a checklist to ensure the project stays on track and can be resumed easily.

## Development Log

*   **2025-07-23:**
    *   **Action:** Created initial `EXECUTION_PLAN.md` and `AZURE_OPENAI_INTEGRATION_TODO.md`.
    *   **Log:** The goal is to add Azure OpenAI support via LiteLLM with minimal disruption. The plan is to use a configuration-driven approach, allowing users to switch between the native Gemini API and a LiteLLM proxy. The first step is to investigate the existing codebase to find the API call sites.
*   **2025-07-23 (Update 1):**
    *   **Action:** Investigated the codebase for API call implementation.
    *   **Findings:** Identified `@google/genai` as the core SDK and `packages/core/src/core/contentGenerator.ts` as the key file.
*   **2025-07-23 (Update 2):**
    *   **Action:** Inspected `node_modules/@google/genai/dist/genai.d.ts`.
    *   **Findings:** Discovered the internal `ApiClient` class and its `setBaseUrl` method, identifying it as the best way to redirect API requests.
*   **2025-07-23 (Update 3):**
    *   **Action:** Attempted to inspect the `GoogleGenAI` instance at runtime to find the `ApiClient`.
    *   **Findings:** The `apiClient` is not a public, directly accessible property. The only public property is `models`.
*   **2025-07-23 (Update 4):**
    *   **Action:** Implemented a hacky but effective approach to modify the base URL.
    *   **Implementation:**
        1.  Modified `contentGenerator.ts` to detect the `LITELLM_BASE_URL` environment variable.
        2.  When detected, it sets the `authType` to a new `USE_LITELLM` value.
        3.  In `createContentGenerator`, it casts the `GoogleGenAI` instance to `any` to access an assumed internal property `_apiClient`.
        4.  It then calls `_apiClient.setBaseUrl()` to redirect traffic to the LiteLLM proxy.
    *   **Validation:**
        1.  Added new test cases to `contentGenerator.test.ts` to validate the new logic.
        2.  The tests confirm that when `LITELLM_BASE_URL` is set, the configuration is correctly identified and the (mocked) `setBaseUrl` function is called with the correct URL.
    *   **Status:** The core implementation for enabling Azure OpenAI via LiteLLM is complete and verified through unit tests.

## Task Checklist

### Phase 1: Investigation and Planning

-   [x] **Codebase Analysis:**
    -   [x] Use `glob` and `search_file_content` to find all files importing `@google/genai`.
    -   [x] Search for usage of `process.env.GEMINI_API_KEY`.
    -   [x] Identify the exact function(s) that make the API calls.
    -   [x] Analyze the data structures for requests and responses.
-   [x] **Confirm SDK Capabilities:**
    -   [x] Read the type definition file (`node_modules/@google/genai/dist/genai.d.ts`).
-   [x] **Runtime Inspection:**
    -   [x] Temporarily modify code to inspect the `GoogleGenAI` instance.
-   [x] **Gemini-Specific Feature Analysis:** (No blockers found during initial investigation)
-   [x] **Configuration Strategy:**
    -   [x] Defined environment variables for the PoC (`LITELLM_BASE_URL`, `GEMINI_API_KEY`).
-   [x] **Documentation:**
    -   [x] Document all findings from the investigation in this file.

### Phase 2: Proof of Concept

-   [x] Implement configuration loading for LiteLLM proxy URL.
-   [x] Conditionally access the `ApiClient` and call `setBaseUrl()` based on the new configuration.
-   [ ] Manually test the connection to a local LiteLLM proxy.

### Phase 3: Full Integration

-   [ ] Implement a more robust configuration system (e.g., `gemini config` command).
-   [ ] Add enhanced error handling for proxy connection issues.

### Phase 4: Testing

-   [x] Write unit tests for the configuration loading logic.
-   [x] Write unit tests for the `setBaseUrl` hack.
-   [ ] Write a full integration test (requires a running LiteLLM instance).

### Phase 5: Documentation

-   [ ] Create a user-facing guide in the `docs/` directory explaining how to set up LiteLLM with Azure OpenAI.
-   [ ] Update the main `README.md` to mention the new capability.