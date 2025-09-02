# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Setup and Installation:**
```bash
# Create virtual environment
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate

# Install in editable mode with dev dependencies
pip install -e .[dev]

# Download required files
python main.py download-files
```

**Running the Agent:**
```bash
# Run in development mode
python main.py dev

# Dispatch an agent (after dev server is running)
lk dispatch create \
  --agent-name base_agent \
  --metadata '{"agent_id": "default_assistant", "call_id": "xyz", "customer_name": "", "customer_id": "xyz", "phone_number": ""}'
```

**Testing:**
```bash
# Run tests
pytest

# Run tests with coverage
pytest --cov

# Run specific test
pytest tests/test_specific.py
```

**Code Quality:**
```bash
# Format code
black .

# Lint code
flake8

# Type checking
mypy
```

**Config Builder (Streamlit):**
```bash
cd configbuilder/streamlit
pip install -r requirements.txt
streamlit run agent_config_builder.py
```

## Architecture Overview

**Vaani** is a configuration-driven voice AI framework built on LiveKit Agents SDK. The system uses JSON configuration files to define agent behavior without code changes.

### Core Architecture

1. **Entry Point**: `main.py` uses LiveKit's CLI to start workers via `universalagent.agents.entrypoint.create_worker_options()`

2. **Agent Configuration**: JSON files in `example/configs/` define complete agent behavior:
   - LLM/TTS/STT provider configuration
   - RAG and memory settings
   - Tool registration
   - Conversation parameters

3. **Component Factory Pattern**: `universalagent.components.factory` creates providers (OpenAI, Deepgram, ElevenLabs, etc.) based on config

4. **Plugin Architecture**: 
   - Built-in tools in `universalagent.tools/`
   - RAG via LlamaIndex + Pinecone when enabled
   - Memory via Mem0 integration when enabled
   - MCP (Model Context Protocol) support via `livekit-agents[mcp]`

### Package Structure

```
universalagent/
├── agents/          # Agent implementations and entrypoint
├── components/      # Component factory for providers
├── core/           # Configuration models and loading
├── events/         # Event handling system
├── kb_retriever/   # Knowledge base retrieval interfaces
├── tools/          # Built-in and extensible tools
└── transcripts/    # Conversation transcription models
```

### Key Components

- **ConfigurableAgent**: Main agent class that orchestrates LLM, TTS, STT based on config
- **ComponentFactory**: Creates provider instances (OpenAI LLM, Deepgram STT, etc.) from config
- **ToolHolder**: Wrapper for exposing Python functions to LLM as tools
- **AgentConfig**: Pydantic model defining complete agent configuration schema

### Configuration-Driven Design

All agent behavior is controlled via JSON configuration files. Key sections:
- `llm_config/tts_config/stt_config`: Provider and model selection
- `rag_config`: Knowledge retrieval pipeline toggle
- `memory_config`: Per-user conversation memory
- `tools`: List of tool names to expose to LLM
- Conversation limits and timeout settings

### Provider Integration

The system supports multiple providers through a unified interface:
- **LLM**: OpenAI, Anthropic, Local models
- **TTS**: ElevenLabs, Cartesia, OpenAI
- **STT**: Deepgram, ElevenLabs, OpenAI
- **RAG**: Pinecone + LlamaIndex
- **Memory**: Mem0 vector-based storage

### Tool System

- Built-in tools: `end_call`, `search_knowledge_base` (when RAG enabled), `get_memory`/`store_important_info` (when memory enabled)
- Custom tools: Wrap async Python functions with `ToolHolder`
- Composio integration for external API access
- Future: OpenAPI/JSON Schema tool definitions

Environment variables (in `.env`) store API keys while configs remain shareable without secrets.