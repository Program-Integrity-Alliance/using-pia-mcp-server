# PIA MCP Examples

This directory contains example scripts demonstrating how to integrate the PIA MCP Server with LangChain and various LLM providers.

You can find out more about PIA's MCP Server [here](https://programintegrity.org/connect/)

## LangChain MCP Integration Example

This example shows how to connect to the PIA MCP Server, discover available tools, and use them with AI agents powered by OpenAI, Azure OpenAI, Claude, or Hugging Face models.

### Features

- **Remote MCP Connection**: Connect to the PIA MCP server via HTTP/SSE using official `langchain-mcp-adapters`
- **Dynamic Tool Discovery**: Automatically discover and load all available MCP tools
- **Multiple LLM Support**: Works with OpenAI, Azure OpenAI, Claude (Anthropic), and Hugging Face models
- **LangChain Integration**: Seamlessly integrate MCP tools with LangChain agents using modern LangChain 1.0 API
- **Example Queries**: Includes working examples for common use cases

### Files in This Directory

- `langchain_mcp_example.py` - Complete working example script
- `requirements.txt` - Python dependencies for the example
- `env.example` - Environment variable template (copy to `.env`)
- `README.md` - This file

**Implementation Note**: This example uses the official `langchain-mcp-adapters` package to connect to remote HTTP/SSE MCP servers. The implementation uses:
- `MultiServerMCPClient` with `streamable_http` transport for remote MCP servers
- `create_agent` from LangChain 1.0 for modern agent creation
- Automatic tool discovery and conversion to LangChain tools
- Works with any tool-calling LLM (OpenAI, Azure, Claude)

## Quick Start

### 1. Set Up Virtual Environment (Recommended)

Using a virtual environment keeps dependencies isolated:

```bash
# Create a virtual environment
python3 -m venv venv

# Activate the virtual environment
# On macOS/Linux:
source venv/bin/activate
# On Windows:
# venv\Scripts\activate

# Your prompt should now show (venv) indicating the venv is active
```

### 2. Install Dependencies

```bash
# Make sure your virtual environment is activated
pip install -r requirements.txt
```

### 3. Configure Environment Variables

```bash
# Copy the example environment file
cp env.example .env

# Edit .env with your actual API keys
# Use your preferred text editor:
nano .env
# or
vim .env
# or
code .env  # VS Code
```

**Required variables:**
- `PIA_API_KEY` - Go to [https://programintegrity.org/](https://programintegrity.org/), register for an account, then you can generate a key under top-right user > API/MCP Keys
- `LLM_PROVIDER` - Choose: `openai`, `azure`, `claude`, or `huggingface`
- Provider-specific API keys (see env.example for details)

You only need to configure one LLM provider, but feel free to set up multiple for comparison.

### 4. Run the Example

```bash
# Make sure your virtual environment is activated
# Run with default example queries:
python langchain_mcp_example.py

# Or run with your own query:
python langchain_mcp_example.py "What is ambulance fraud?"

# Another example:
python langchain_mcp_example.py "Find recent cybersecurity reports from OIGs"
```

```

## LLM Provider Setup

Choose one of the following providers and configure the corresponding environment variables in your `.env` file:

### OpenAI (Default)

```bash
LLM_PROVIDER=openai
OPENAI_API_KEY=sk-...your-openai-api-key...
OPENAI_MODEL=gpt-4  # optional, default: gpt-4
```

**Get API Key:** https://platform.openai.com/api-keys

### Azure OpenAI

```bash
LLM_PROVIDER=azure
AZURE_OPENAI_API_KEY=your-azure-api-key
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=your-deployment-name
AZURE_OPENAI_API_VERSION=2024-02-15-preview  # optional
```

**Setup Requirements:**
1. Azure subscription
2. Azure OpenAI resource created in Azure Portal
3. Model deployment (GPT-5, GPT-5-mini, etc.)
4. Get credentials from: https://portal.azure.com

### Claude (Anthropic)

```bash
LLM_PROVIDER=anthropic
ANTHROPIC_API_KEY=sk-ant-...your-anthropic-api-key...
ANTHROPIC_MODEL=claude-sonnet-4-5-20250929  # optional
```

**Get API Key:** https://console.anthropic.com/

### Hugging Face

```bash
LLM_PROVIDER=huggingface
# Use either HF_TOKEN (preferred) or HUGGINGFACE_API_KEY
HF_TOKEN=hf_...your-huggingface-token...
# Default model: Qwen/Qwen2.5-72B-Instruct
HUGGINGFACE_MODEL=Qwen/Qwen2.5-72B-Instruct
```

**Get API Key:** https://huggingface.co/settings/tokens

**Important:** The HuggingFace model you choose **must support function calling/tool use** for agents to work properly. Recommended models that support function calling:
- `Qwen/Qwen2.5-72B-Instruct` (default, excellent performance)
- `Qwen/Qwen2.5-7B-Instruct` (smaller, faster)
- `mistralai/Mixtral-8x7B-Instruct-v0.1`

Note: Many Llama models do not support function calling through the HuggingFace Router API.

## What the Example Does

1. **Connects to PIA MCP Server** - Establishes an HTTP connection to https://www.programintegrity.org/mcp
2. **Initializes MCP Session** - Performs MCP protocol handshake
3. **Discovers Tools** - Retrieves the available PIA tools (`pia_search` across every source — GAO, OIG, CRS, DOJ, Congress.gov, Federal Register — plus `pia_oversight_recommendations`)
4. **Converts to LangChain Tools** - Wraps each MCP tool as a LangChain `StructuredTool`
5. **Creates AI Agent** - Sets up a LangChain agent with your chosen LLM
6. **Executes Queries** - Agent automatically selects and calls appropriate tools to answer questions

## Available MCP Tools

The script automatically discovers whatever tools the PIA MCP server currently
offers at runtime, so it stays correct as the tool set evolves. Today the server
exposes two tools:

- **`pia_search`** — Search the full PIA database of government oversight reports,
  recommendations, executive orders, legislation, and integrity data (GAO,
  Oversight.gov/OIG, CRS, DOJ, Congress.gov, Federal Register). A single entry
  point for all document search:
  - **Scope** by source, dataset, agency, or date with the `filter` parameter (an
    OData expression, e.g. `SourceDocumentDataSource eq 'GAO'`). This replaces the
    former per-source tools — to search just one source, pass its
    `SourceDocumentDataSource` filter.
  - Choose full-text **content** vs. document **titles** with `search_mode`
    (`content` or `titles`). Counts in `content` mode are text *chunks*, not whole
    documents; use `titles` when you need a document/report count.
  - Sweep **every source at once** with `wide=true`, or discover the available
    filter values with `facets_only=true`.

- **`pia_oversight_recommendations`** — Search the Open Recommendations dataset
  (GAO and Oversight.gov/OIG only), with facet breakdowns (status, priority,
  agency, theme, GAO topic) enabled by default. Returns the count of open
  recommendations plus a Rec Spotlight link for the full result set.

> **Migration note:** PIA previously exposed a set of narrower tools
> (`pia_search_content`, `pia_search_titles`, `pia_search_content_gao`,
> `pia_search_content_oig`, `pia_search_content_crs`, `referenced_agencies`, …).
> These have been **consolidated into `pia_search`** — use its `filter`,
> `search_mode`, `wide`, and `facets_only` parameters instead of the old
> per-source tools. If your code hard-coded the old names, switch to `pia_search`.

## Example Queries

```bash
# Search for fraud-related recommendations
python langchain_mcp_example.py "Find open recommendations about fraud from GAO"

# Search executive orders
python langchain_mcp_example.py "What executive orders address artificial intelligence?"

# Search recent reports
python langchain_mcp_example.py "Show me the latest cybersecurity reports from OIG"

# Search across multiple sources
python langchain_mcp_example.py "Find recommendations about improper payments"

# Get specific information
python langchain_mcp_example.py "What agencies are most mentioned in DOJ documents?"
```

## Troubleshooting

### Virtual Environment Issues

**Problem:** `python3: command not found`
```bash
# Try using 'python' instead of 'python3':
python -m venv venv
```

**Problem:** Virtual environment not activating
```bash
# Make sure you're in the examples directory
pwd  # Should show: .../pia-mcp/examples

# Try activating again:
source venv/bin/activate

# Check if it worked (you should see (venv) in your prompt):
which python  # Should show: .../examples/venv/bin/python
```

### Dependency Installation Issues

**Problem:** `pip: command not found`
```bash
# Make sure your virtual environment is activated
# Then try:
python -m pip install -r requirements.txt
```

**Problem:** Permission denied errors
```bash
# This shouldn't happen with venv, but if it does:
# Make sure your virtual environment is activated (you should see (venv) in prompt)
# If still having issues, recreate the venv:
rm -rf venv
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### API Key Errors

**Problem:** `PIA_API_KEY environment variable is required`
```bash
# Make sure you created and edited the .env file:
ls -la .env  # File should exist
cat .env     # Should show your configuration

# Make sure you're running from the examples directory:
pwd  # Should show: .../pia-mcp/examples
```

**Problem:** `OPENAI_API_KEY environment variable is required`
- Verify you set the correct API key for your chosen `LLM_PROVIDER` in `.env`
- For Azure, you need `AZURE_OPENAI_API_KEY`, not `OPENAI_API_KEY`
- For Claude, you need `ANTHROPIC_API_KEY`

### Connection Issues

**Problem:** `Connection timeout` or `Connection refused`
```bash
# Check your internet connection, then verify the server is reachable.
# A POST without an API key should return HTTP 403 with an
# "API key required in x-api-key header" message — that response confirms
# connectivity (you just need a valid key).
curl -sS -X POST https://www.programintegrity.org/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}'

# Verify the PIA_MCP_URL in your .env file:
cat .env | grep PIA_MCP_URL
# Should show: PIA_MCP_URL=https://www.programintegrity.org/mcp
```

**Problem:** `403` / `API key required`
- Your `PIA_API_KEY` is missing or invalid. Generate a fresh key at
  https://www.programintegrity.org (account menu, top-right → **API/MCP key**)
  and make sure it's set in `.env`.
- Use the `www.` host exactly — `https://www.programintegrity.org/mcp`. The bare
  apex `programintegrity.org/mcp` issues a redirect that strips the `x-api-key`
  header, so auth will fail.

### Import Errors

**Problem:** `ModuleNotFoundError: No module named 'langchain'`
```bash
# Make sure your virtual environment is activated
source venv/bin/activate

# Reinstall dependencies
pip install -r requirements.txt

# Verify installation
pip list | grep langchain
```

## Advanced Usage

### Using Different Models

Edit your `.env` file to use different models:

```bash
# OpenAI - use GPT-4 Turbo
OPENAI_MODEL=gpt-4-turbo

# Claude - use a different Anthropic model
ANTHROPIC_MODEL=claude-opus-4-1-20250805

# Hugging Face - use Mistral
HUGGINGFACE_MODEL=mistralai/Mistral-7B-Instruct-v0.2
```

### Switching Between Providers

You can quickly switch providers by changing one line in `.env`:

```bash
# To use OpenAI:
LLM_PROVIDER=openai

# To use Azure:
LLM_PROVIDER=azure

# To use Claude:
LLM_PROVIDER=anthropic
```

Then run the script again - no code changes needed!

### Customizing the Agent

Edit `langchain_mcp_example.py` to customize:
- Agent system prompt (line ~342)
- Temperature and other LLM parameters (lines ~253-306)
- Tool selection and filtering (lines ~233-238)

## Getting API Keys

### PIA MCP Server (Required)
Register for a free PIA Community account at https://www.programintegrity.org,
then generate a key from the account menu (top-right) → **API/MCP key**. Send it
in the `x-api-key` header (or as a Bearer token) when connecting to
`https://www.programintegrity.org/mcp`. See
https://www.programintegrity.org/connect for a full walkthrough.

### LLM Providers (Choose One)
- **OpenAI**: https://platform.openai.com/api-keys
- **Azure OpenAI**: https://portal.azure.com (requires Azure subscription)
- **Claude/Anthropic**: https://console.anthropic.com/
- **Hugging Face**: https://huggingface.co/settings/tokens

## Complete Setup Example (Step-by-Step)

Here's a complete walkthrough from scratch:

```bash
# 1. Navigate to examples directory
cd /path/to/pia-mcp/examples

# 2. Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate  # You should see (venv) in your prompt

# 3. Install dependencies
pip install -r requirements.txt

# 4. Copy environment template
cp env.example .env

# 5. Edit .env file (use your preferred editor)
nano .env
# Add your PIA_API_KEY
# Choose your LLM_PROVIDER
# Add the corresponding API key

# 6. Save and close the editor (Ctrl+X, then Y, then Enter for nano)

# 7. Test the setup
python langchain_mcp_example.py "Test query: find fraud reports"

# 8. When done, deactivate virtual environment
deactivate
```

## Support

- **Main Documentation**: See the [PIA Connect page](https://www.programintegrity.org/connect) for full MCP setup instructions
- **Issues**: Open an issue on the GitHub repository
- **Questions**: Check the troubleshooting section above

