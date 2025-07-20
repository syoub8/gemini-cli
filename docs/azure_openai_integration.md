# Guide: Using `gemini-cli` with Azure OpenAI

## Introduction

This guide explains how to use Azure OpenAI API as the backend for `gemini-cli` instead of the Google Gemini API.
By using [LiteLLM](https://github.com/BerriAI/litellm) as a proxy, this integration can be achieved without any code changes to `gemini-cli` itself.

This document provides the best practices for an **Azure OpenAI-only environment**.

## Prerequisites

- **Node.js / npm**: Required to run `gemini-cli`.
- **Python / pip**: Required to run `litellm`.
- **`@google/gemini-cli`**: Should be installed (e.g., `npm install -g @google/gemini-cli`).
- **Azure OpenAI Credentials**:
    - API Key
    - Endpoint URL
    - Deployed model names for reasoning (`o4-mini`, `o3`, etc.), fast (`gpt-4o-mini`), and embedding models.

## Step 1: Configure LiteLLM Proxy

LiteLLM acts as the brain's command center, translating and directing requests from `gemini-cli` to the correct Azure OpenAI models.

1.  **Install LiteLLM**:
    Install LiteLLM with proxy support by running the following command in your terminal:
    ```bash
    pip install 'litellm[proxy]'
    ```

2.  **Review the Configuration File**:
    The `litellm_config.yaml` file in the project root is pre-configured with recommended settings. You only need to replace the placeholders with your specific Azure OpenAI deployment details.

    **Note for Corporate Proxy Users**: If your system uses a corporate proxy (via `HTTPS_PROXY` environment variable), LiteLLM will automatically use it for its outbound requests to Azure. No extra configuration is needed in this file for LiteLLM to work.

## Step 2: Configure `gemini-cli` for Azure Context

To ensure `gemini-cli`'s UI and features (like context window management) work correctly with your Azure models, you need to inform it of their context lengths.

-   Open or create your `settings.json` file (usually located at `~/.gemini/settings.json`).
-   Add the `modelContextWindowMap` setting as shown below. This tells `gemini-cli` the correct context window size for the models you are using via the proxy.

    ```json:~/.gemini/settings.json
    {
      "model": "gemini-2.5-pro",
      "modelContextWindowMap": {
        "gemini-2.5-pro": 200000,
        "gemini-2.5-flash": 128000
      }
    }
    ```
    **Note**: The keys (`gemini-2.5-pro`) should match what `gemini-cli` uses, and the values (`200000`) should match the context window of your actual Azure model (`o4-mini` in this case).

## Step 3: Launch and Run

Now, you can launch the LiteLLM proxy and run `gemini-cli`.

1.  **Set Azure API Key**:
    ```bash
    export AZURE_API_KEY="<your_azure_api_key>"
    ```

2.  **Start LiteLLM Proxy**:
    Run this command from the project's root directory. It's recommended to run it in the background (`&`).
    ```bash
    litellm --config litellm_config.yaml &
    ```

3.  **Set Dummy Gemini API Key**:
    The `GEMINI_API_KEY` is required by the CLI, but the value can be a dummy string as authentication is handled by LiteLLM.
    ```bash
    export GEMINI_API_KEY="sk-dummy-key-for-azure"
    ```

4.  **Run `gemini-cli` using the `--proxy` flag**:
    This is the crucial step. Use the `--proxy` flag to direct `gemini-cli`'s traffic to the local LiteLLM instance. This flag overrides any system-wide `HTTPS_PROXY` settings for `gemini-cli` only.

    **For interactive mode:**
    ```bash
    gemini --proxy "http://127.0.0.1:4000"
    ```

    **For non-interactive mode:**
    ```bash
    gemini --proxy "http://127.0.0.1:4000" "What are the key features of the Azure o-series models?"
    ```

---

## Troubleshooting SSL Errors in Corporate Environments
If your corporate proxy performs SSL/TLS inspection, you might encounter certificate errors. To resolve this, you may need to tell Python (which runs LiteLLM) to trust your company's root CA certificate.

Set the following environment variable before starting LiteLLM:
```bash
export SSL_CERT_FILE=/path/to/your/corporate-ca-bundle.crt
litellm --config litellm_config.yaml &
```

---

## Understanding Feature Differences

When using this setup, it's important to be aware of the following:
- **"Thinking" Summaries are Not Available**: The UI feature in `gemini-cli` that shows the model's thought process is specific to the Gemini API and will not be displayed.
- **Error Messages**: Errors will originate from Azure OpenAI and be relayed through LiteLLM, so they may differ from native Gemini errors.
- **Model Behavior**: Different models have different "personalities." You may need to adjust your prompts to get the best results from the `o`-series or GPT models.