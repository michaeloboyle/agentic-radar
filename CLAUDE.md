# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Setup and Dependencies
```bash
# Install dependencies with Poetry
poetry install

# Install with specific framework extras
poetry install -E crewai        # For CrewAI tools (Python 3.10-3.12 only)
poetry install -E openai-agents # For OpenAI Agents testing

# Activate virtual environment
poetry shell

# Add development dependencies
poetry install --with dev
```

### CLI Usage
```bash
# Run current development version
python agentic_radar/cli.py --help
python -m agentic_radar --help

# Scan agentic workflows
agentic-radar scan langgraph -i path/to/code -o report.html
agentic-radar scan crewai --harden-prompts -i examples/crewai/ -o report.html
agentic-radar scan openai-agents -i examples/openai-agents/ -o report.html

# Graph-only visualization (minimal output)
agentic-radar scan langgraph --graph-only -i path/to/code -o graph.html

# Test workflows for vulnerabilities (requires OPENAI_API_KEY)
agentic-radar test openai-agents "path/to/script.py"
agentic-radar test openai-agents --config custom_tests.yaml "script.py"
```

### Testing and Quality
```bash
# Run tests
pytest                            # Run all tests
pytest tests/cli_test.py          # Run specific test file
pytest tests/unit_tests/          # Run unit tests only
pytest -v                         # Verbose output
pytest --tb=short                 # Short traceback format

# Browser testing (optional)
pytest -m browser                 # Run browser tests only
pytest -m "not browser"          # Skip browser tests (faster development)
pip install pytest-playwright    # Install browser testing dependencies
playwright install               # Install browser binaries

# Run pre-commit checks (linting, formatting, type checking)
pre-commit run --all-files

# Individual quality checks
ruff check agentic_radar/         # Lint
ruff format agentic_radar/        # Format
mypy agentic_radar/              # Type check

# Poetry-specific commands
poetry run pytest               # Run tests in Poetry environment
poetry run pre-commit run --all-files
```

### Environment Variables
Copy `.env.example` to `.env` and configure:
- `OPENAI_API_KEY` - Required for prompt hardening and testing features
- `AZURE_OPENAI_API_KEY` / `AZURE_OPENAI_ENDPOINT` - Alternative to OpenAI API

## Project Timeline
See `docs/project-gantt.md` for the complete development timeline and roadmap.

## Architecture

### Core Components

**Analysis Engine (`agentic_radar/analysis/`)**
- Framework-specific analyzers inherit from `Analyzer` base class
- Each framework (langgraph, crewai, n8n, openai-agents, autogen) has its own analysis module
- Analyzers parse source code to extract agents, tools, workflows, and MCP servers
- Output: `GraphDefinition` objects representing the agentic workflow

**CLI Interface (`agentic_radar/cli.py`)**
- Two main commands: `scan` (static analysis) and `test` (runtime vulnerability testing)
- Uses Typer for command-line interface
- Coordinates analysis, vulnerability mapping, prompt hardening, and report generation

**Vulnerability Mapping (`agentic_radar/mapper/`)**
- Maps detected tools to known security vulnerabilities
- Uses `vulnerabilities.json` database aligned with OWASP frameworks
- Provides security risk assessment for identified components

**Testing Framework (`agentic_radar/test/`)**
- Runtime vulnerability testing with adversarial inputs
- Agent adapters for different frameworks
- Oracle-based evaluation using LLM to assess test results
- Rich terminal output for test results

**Report Generation (`agentic_radar/report/`)**
- Jinja2 templates for HTML reports
- Graph visualization of agentic workflows
- Comprehensive security and operational insights

**Prompt Hardening (`agentic_radar/prompt_hardening/`)**
- Pipeline-based system for improving agent prompts
- OpenAI integration for prompt enhancement
- PII protection and security-focused improvements

### Framework Support Matrix

| Feature               | LangGraph | CrewAI | n8n | OpenAI Agents | Autogen |
|-----------------------|-----------|--------|-----|---------------|---------|
| Workflow Scanning     | ✅         | ✅      | ✅   | ✅             | ✅       |
| MCP Server Detection  | ✅         | ❌      | ❌   | ✅             | ❌       |
| Prompt Hardening      | ❌         | ✅      | ❌   | ✅             | ✅       |
| Runtime Testing       | ❌         | ❌      | ❌   | ✅             | ❌       |

### Adding New Framework Support

1. Create analyzer module in `agentic_radar/analysis/{framework}/`
2. Implement `Analyzer` interface with `analyze()` method
3. Add framework enum to `cli.py` 
4. Register analyzer in `agentic_radar/analysis/__init__.py`
5. Add optional dependencies to `pyproject.toml` if needed

### Key Data Models

- `GraphDefinition` - Complete workflow representation
- `Agent` - Individual agent with tools, prompts, and metadata
- `Tool` - External or custom tool with security classification
- `Test` - Vulnerability test case with input and success conditions

## Testing Standards

Agentic Radar uses a **tiered testing approach** for HTML+JavaScript reports:

### Tier 1: Python Unit Tests (Required)
- Test HTML generation, data serialization, CLI behavior
- Use pytest markers: `@pytest.mark.supported` and `@pytest.mark.not_supported`
- Framework analysis tests verify parsing of known code patterns
- CLI tests validate command interfaces and error handling
- Runtime tests require API keys and may make external calls

### Tier 2: Browser Integration Tests (Optional)
- Test JavaScript functionality and user interactions
- Use `@pytest.mark.browser` marker to separate from unit tests
- Test graph rendering, layout switching, parameter controls
- Install with: `pip install pytest-playwright && playwright install`
- Run with: `pytest -m browser` or skip with: `pytest -m "not browser"`

See `docs/testing-standards.md` for complete testing guidelines and best practices.

## Security Considerations

- Code analysis is performed locally - no source code is shared externally
- Optional features (prompt hardening, testing) may send data to LLM providers
- Vulnerability database focuses on defensive security analysis
- Test configurations can include custom attack vectors via YAML files