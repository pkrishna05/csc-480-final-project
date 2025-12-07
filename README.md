# LLM-Mutation-Guided Testing

An advanced mutation testing framework that uses Large Language Models (LLMs) to generate subtle code mutants and automatically create test cases that expose those mutants. This tool is designed to improve test coverage by identifying gaps in existing test suites through AI-driven mutation analysis.

**Reference Paper:** [https://arxiv.org/pdf/2501.12862](https://arxiv.org/pdf/2501.12862)

## Overview

This repository implements an LLM-driven mutation testing workflow that:

1. **Chunks** Python code files into logical, mutable units (functions, methods, classes)
2. **Generates** subtle mutants for each chunk using LLM guidance
3. **Validates** mutants to ensure they build and pass existing tests
4. **Detects** equivalent mutants (mutants that behave identically to the original)
5. **Creates** targeted test cases that kill non-equivalent mutants
6. **Evaluates** mutant quality using LLM-based scoring
7. **Processes** multiple chunks in parallel for efficiency

The tool is particularly effective at finding subtle bugs related to privacy violations, security issues, and business logic flaws that existing tests might miss.

## Features

- **Dual Chunking Modes**: 
  - LLM-based chunking for intelligent code segmentation
  - AST-based chunking for fast, deterministic parsing
- **Parallel Processing**: Process multiple code chunks simultaneously
- **Smart Deduplication**: Avoids generating duplicate mutants
- **Comprehensive Validation**: Multi-step validation ensures mutants are valid and meaningful
- **Quality Scoring**: LLM-based evaluation of mutant quality across multiple dimensions
- **Repository Management**: Handles dependencies and test environments automatically
- **Caching**: Caches chunking results to speed up repeated runs

## Architecture

The workflow consists of several key components:

- **CodeChunker**: Extracts mutable code chunks from Python files (LLM or AST mode)
- **LLMOrchestrator**: Manages all LLM interactions (mutant generation, test generation, equivalence detection, quality scoring)
- **MutationPipeline**: Executes the 7-step mutation testing pipeline for each chunk
- **ParallelProcessor**: Handles parallel execution across multiple chunks
- **WorkflowOrchestrator**: Coordinates the overall workflow
- **RepoManager**: Manages temporary repository copies for isolated testing
- **CodeValidator**: Validates syntax, builds, and test execution

## Prerequisites

- **Python 3.8+**
- **API Keys**: 
  - `GEMINI_API_KEY` and `GOOGLE_API_KEY` for Google's Gemini models
  - Configured in your environment or `.env` file

## Installation

1. **Clone the repository**:
   ```bash
   git clone <repository-url>
   cd csc-480-final-project
   ```

2. **Create and activate a virtual environment**:
   ```bash
   # Windows (PowerShell)
   python -m venv .venv
   .\.venv\Scripts\activate
   
   # macOS/Linux
   python -m venv .venv
   source .venv/bin/activate
   ```

3. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

4. **Set up environment variables**:
   Create a `.env` file in the project root:
   ```bash
   GEMINI_API_KEY=your_api_key_here
   GOOGLE_API_KEY=your_api_key_here
   ```

## Usage

### Basic Usage

Run the main script with the repository path, code file, and test file:

```bash
python main.py <repo_path> <code_file> <test_file> [max_workers] [chunker_mode]
```

### Arguments

- **`repo_path`**: Path to repository root (e.g., `.` for current directory or `/path/to/repo`)
- **`code_file`**: Path to the Python code file to mutate (relative to repo or absolute)
- **`test_file`**: Path to the existing test file (relative to repo or absolute)
- **`max_workers`** (optional): Number of parallel workers (default: 3)
- **`chunker_mode`** (optional): Chunking mode - `'llm'` or `'ast'` (default: `'llm'`)

### Examples

**Example 1: Simple usage with default settings**
```bash
python main.py . examples/simple_example.py examples/simple_example_test.py
```

**Example 2: Using AST chunking with more workers**
```bash
python main.py . src/validators.py tests/test_validators.py 5 ast
```

**Example 3: Absolute paths**
```bash
python main.py /path/to/repo /path/to/repo/src/file.py /path/to/repo/tests/test_file.py
```

## Workflow Steps

The mutation testing pipeline follows these steps for each code chunk:

1. **Generate Mutant**: LLM creates a subtle bug in the code chunk
2. **Syntactic Identity Check**: Ensures the mutant is syntactically different
3. **Build & Test Validation**: Verifies the mutant builds and passes existing tests
4. **Equivalence Detection**: Checks if the mutant behaves identically to the original
5. **Test Generation**: LLM creates a test case that kills the mutant
6. **Test Validation**: 
   - Verifies the new test passes on the original code
   - Verifies the new test fails on the mutant
7. **Quality Scoring**: LLM evaluates the mutant and test quality across multiple dimensions

## Output Structure

Results are saved in the `outputs/` directory, organized by code file name:

```
outputs/
└── <code_file_name>/
    ├── metadata.json              # Metadata about all generated mutants
    ├── mutant_<chunk_id>_<hash>.py # Mutated code files
    └── test_<chunk_id>_<hash>.py   # Generated test files
```

### Metadata Format

The `metadata.json` file contains:
```json
{
  "code_file": "/path/to/code/file.py",
  "total_chunks": 5,
  "successful_count": 3,
  "mutants": [
    {
      "hash": "abc123def456",
      "chunk_id": "validate_email",
      "chunk_type": "function",
      "files": {
        "mutant": "mutant_validate_email_abc123def456.py",
        "test": "test_validate_email_abc123def456.py"
      },
      "scores": {
        "concern_alignment": 8,
        "business_logic_impact": 7,
        "mutation_subtlety": 9,
        "test_effectiveness": 8,
        "test_integration": 7
      }
    }
  ]
}
```

## Configuration

### Model Configuration

Edit `constants.py` to change default settings:

```python
MODEL = "gemini-2.5-flash"        # LLM model name
MODEL_PROVIDER = "google_genai"    # Model provider
OUTPUT_DIR = "outputs"             # Output directory
MAX_WORKERS = 3                    # Default parallel workers
```

### Chunking Modes

- **LLM Mode** (`chunker_mode='llm'`): 
  - Uses LLM to intelligently segment code
  - Better at understanding code semantics
  - Results are cached for performance
  - Slower but more context-aware

- **AST Mode** (`chunker_mode='ast'`):
  - Uses Python's AST parser for deterministic chunking
  - Faster execution
  - Less context-aware but more reliable
  - Good for well-structured code

## Quality Scoring

Each generated mutant is evaluated by an LLM judge across five dimensions (0-10 scale):

1. **Concern Alignment**: How well the mutation matches the target violation pattern
2. **Business Logic Impact**: Real-world significance of the bug
3. **Mutation Subtlety**: How likely existing tests would miss this bug
4. **Test Effectiveness**: How well the generated test catches the mutant
5. **Test Integration**: How well the test fits the existing test suite

## Examples

The `examples/` directory contains sample code and tests:

- `simple_example.py` / `simple_example_test.py`: Basic validation class example
- `new_example.py` / `new_example_tests.py`: Additional examples

Run the examples:
```bash
python main.py . examples/simple_example.py examples/simple_example_test.py
```

## Troubleshooting

### Common Issues

1. **API Key Not Found**:
   - Ensure `GEMINI_API_KEY` or `GOOGLE_API_KEY` is set in your environment or `.env` file

2. **Import Errors**:
   - The tool creates isolated test environments, but ensure all dependencies are listed in your project's requirements

3. **Chunking Failures**:
   - Try switching to AST mode if LLM chunking fails: `chunker_mode='ast'`
   - Check that your code file is valid Python

4. **Test Execution Failures**:
   - Ensure your test file uses a compatible test framework (unittest, pytest, etc.)
   - Check that test dependencies are available


## Contributing

This is a research project. For questions or issues, please contact the maintainers on github.