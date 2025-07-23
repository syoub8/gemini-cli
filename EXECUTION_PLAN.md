# Azure OpenAI Integration Execution Plan

## 1. Objective

Modify the Gemini CLI to support using Azure OpenAI as an alternative backend through the LiteLLM proxy. This will allow users to leverage Azure's models while maintaining the existing Gemini CLI functionality and user experience.

## 2. Guiding Principles

- **Minimize Disruption:** Avoid breaking changes to the existing codebase. The default behavior should remain unchanged (using the native Gemini API).
- **Maintainability:** The new code should be clean, well-documented, and easy to maintain.
- **Configurability:** Users should be able to easily switch between the native Gemini API and the Azure OpenAI backend via configuration.
- **Leverage LiteLLM:** Use LiteLLM as the translation layer, as requested. The CLI's responsibility is to correctly route requests to the LiteLLM proxy, not to implement the translation logic itself.

## 3. Phased Approach

The integration will be carried out in distinct phases to ensure a structured and manageable development process.

### Phase 1: Investigation and Planning (Current Phase)

- **Analyze Codebase:** Identify the key files and functions responsible for making API calls to the Google Gemini service. Pay close attention to how requests are constructed and how responses are parsed.
- **Identify Gemini-Specific Features:** Determine if the CLI uses any Gemini-specific features (e.g., specific tool-use schemas, system instructions, safety settings) that might not be directly translatable by LiteLLM.
- **Define Configuration Strategy:** Decide how users will configure the Azure OpenAI settings (e.g., via environment variables, a new section in a configuration file).
- **Finalize Plan:** Refine this execution plan and the initial `AZURE_OPENAI_INTEGRATION_TODO.md`.

### Phase 2: Proof of Concept - Configuration-Based Integration

- **Implement Configuration Loading:** Add logic to the CLI to read the new Azure/LiteLLM configuration (e.g., `LITELLM_PROXY_URL`, `AZURE_API_KEY`, etc.).
- **Conditional API Client Initialization:** Based on the loaded configuration, conditionally modify the API client's base URL and authentication headers to point to the LiteLLM proxy instead of the default Google endpoint.
- **Manual Testing:** Manually run a LiteLLM proxy instance configured for Azure OpenAI. Test if the CLI can successfully send a request through the proxy and receive a valid response. This phase focuses on validating the approach with minimal code changes.

### Phase 3: Full Integration and User Experience

- **Refine Configuration:** Create a more robust and user-friendly configuration system. This might involve adding a `gemini config` command or a dedicated section in a JSON/YAML config file.
- **Error Handling:** Implement specific error handling for LiteLLM/Azure-related issues (e.g., invalid proxy URL, authentication errors with the proxy).
- **Model Mapping:** If necessary, add logic to handle the mapping between Gemini model names used in the CLI and the corresponding model names required by the LiteLLM proxy's `model_group_alias`.

### Phase 4: Testing and Validation

- **Unit/Integration Tests:** Write new tests to cover the Azure OpenAI integration path. This includes testing the configuration loading, conditional logic, and response handling.
- **Regression Testing:** Ensure that all existing tests for the native Gemini functionality continue to pass. The new changes should not introduce any regressions.

### Phase 5: Documentation

- **Update README:** Add a new section to the `README.md` or a new document in the `docs/` directory explaining how to configure and use the Azure OpenAI integration.
- **Provide Examples:** Include example configurations for both the Gemini CLI and the LiteLLM proxy.

## 4. Initial Steps

1.  Create and commit `EXECUTION_PLAN.md` and `AZURE_OPENAI_INTEGRATION_TODO.md`.
2.  Begin **Phase 1** by searching the codebase for `@google/generative-ai` and `GEMINI_API_KEY` to locate the core API interaction logic.
3.  Document findings in `AZURE_OPENAI_INTEGRATION_TODO.md`.
