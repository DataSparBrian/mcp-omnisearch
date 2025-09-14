# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

**Build and Development:**
- `pnpm run build` - Build the TypeScript project
- `pnpm run start` - Run the built server
- `pnpm run dev` - Run via MCP Inspector for debugging
- `pnpm run format` - Format code with Prettier
- `pnpm run format:check` - Check formatting without modifying files

**Testing and Validation:**
- `npx @modelcontextprotocol/inspector dist/index.js` - Interactive tool testing via MCP Inspector
- Build first, then test individual provider tools through the inspector

## Architecture Overview

**MCP Omnisearch** is a Model Context Protocol (MCP) server that provides unified access to multiple search providers and AI tools through a single interface.

### Core Architecture

The codebase follows a provider-based architecture with clear separation of concerns:

**Entry Point (`src/index.ts`):**
- Creates `OmnisearchServer` class that orchestrates the entire system
- Validates environment configuration, initializes providers, registers tools
- Uses `tmcp` library with `ValibotJsonSchemaAdapter` for MCP protocol handling

**Provider System (`src/providers/`):**
- **Search providers**: Tavily, Brave, Kagi, GitHub, Exa
- **AI Response providers**: Perplexity, Kagi FastGPT, Exa Answer
- **Processing providers**: Jina Reader, Firecrawl (scrape/crawl/map/extract/actions), Tavily Extract, Exa Contents/Similar, Kagi Summarizer
- **Enhancement providers**: Jina Grounding, Kagi Enrichment

**Tool Registration (`src/server/tools.ts`):**
- `ToolRegistry` class manages all provider registration and tool creation
- Dynamically registers tools based on available API keys
- Creates MCP tool schemas using Valibot validation
- Special handling for GitHub provider (separate tools for code/repository/user search)

**Configuration (`src/config/env.ts`):**
- Centralized API key and provider configuration
- Flexible validation that enables only providers with valid API keys
- Provider-specific timeouts and base URLs

### Key Conventions

**HTTP Requests:**
- Always use `src/common/http.ts` (`http_json`) for all API calls
- Never use raw `fetch` in provider code
- Includes standardized error handling and retry logic with `ProviderError`

**API Key Management:**
- All keys read from environment variables via `src/config/env.ts`
- Providers are opt-in - missing keys disable that provider's tools
- Use `is_api_key_valid()` to check key availability before registration

**Error Handling:**
- Use `ProviderError` with proper error types (API_ERROR, PROVIDER_ERROR, etc.)
- `http_json` handles common HTTP errors (401/403/429/5xx) consistently
- Include request timeouts using `AbortSignal.timeout()` from config

**Tool Patterns:**
- Search tools: query + optional limit/include_domains/exclude_domains
- Processing tools: url(s) + optional extract_depth (basic/advanced)
- Enhancement tools: content parameter

The server automatically detects available API keys on startup and only registers tools for providers with valid credentials, allowing flexible deployment with any subset of supported services.