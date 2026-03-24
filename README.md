# AI Testing MCP

[![Built by Groovy Web](https://img.shields.io/badge/Built%20by-Groovy%20Web-0f3460?logo=github&logoColor=white)](https://www.groovyweb.co/?utm_source=github&utm_medium=readme&utm_campaign=ai-testing-mcp)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Model Context Protocol (MCP) server for comprehensive AI testing, evaluation, and quality assurance.

## Overview

AI Testing MCP provides standardized testing methodologies, evaluation metrics, and automated testing workflows for AI/ML systems. It implements the Model Context Protocol for seamless integration with AI development tools.

## Features

### Testing Capabilities
- **Unit Tests**: Component-level testing
- **Integration Tests**: End-to-end workflow testing
- **Performance Tests**: Latency, throughput, resource usage
- **Security Tests**: Adversarial attacks, prompt injection
- **Quality Tests**: Output validation, consistency checks

### Evaluation Metrics
- **Accuracy**: Precision, recall, F1 score
- **Quality**: Coherence, relevance, fluency
- **Safety**: Toxicity, bias detection
- **Performance**: Response time, token usage
- **Cost**: API costs, compute resources

### MCP Integration
- **Standard Protocol**: Implements MCP specification
- **Tool Definitions**: Testing tools for AI assistants
- **Resource Management**: Test data and configuration
- **Prompt Templates**: Reusable test prompts

## Quick Start

```bash
# Clone the repository
git clone https://github.com/groovy-web/ai-testing-mcp.git
cd ai-testing-mcp

# Install dependencies
npm install

# Configure environment
cp .env.example .env
# Edit with your API keys

# Start the MCP server
npm start

# Use with MCP client
# Add server configuration to your MCP client
```

## MCP Server Configuration

```json
{
  "mcpServers": {
    "ai-testing": {
      "command": "node",
      "args": ["/path/to/ai-testing-mcp/dist/index.js"],
      "env": {
        "OPENAI_API_KEY": "your-key",
        "ANTHROPIC_API_KEY": "your-key"
      }
    }
  }
}
```

## Available Tools

### Testing Tools

#### `run_test_suite`
Execute a comprehensive test suite for an AI model.

```typescript
{
  "name": "run_test_suite",
  "description": "Run tests on an AI model",
  "inputSchema": {
    "type": "object",
    "properties": {
      "model": { "type": "string" },
      "testCategory": {
        "type": "string",
        "enum": ["accuracy", "performance", "security", "quality"]
      },
      "testCases": { "type": "array" }
    }
  }
}
```

#### `evaluate_output`
Evaluate AI model outputs against metrics.

```typescript
{
  "name": "evaluate_output",
  "description": "Evaluate model output quality",
  "inputSchema": {
    "type": "object",
    "properties": {
      "output": { "type": "string" },
      "expected": { "type": "string" },
      "metrics": { "type": "array" }
    }
  }
}
```

#### `generate_test_cases`
Generate test cases for specific scenarios.

```typescript
{
  "name": "generate_test_cases",
  "description": "Generate test cases",
  "inputSchema": {
    "type": "object",
    "properties": {
      "scenario": { "type": "string" },
      "count": { "type": "number" }
    }
  }
}
```

## Repository Structure

```
ai-testing-mcp/
├── docs/                    # Documentation
│   ├── mcp-protocol.md
│   ├── testing-guide.md
│   └── metrics.md
├── examples/                # Usage examples
│   ├── basic-testing/
│   ├── custom-metrics/
│   └── integration-examples/
├── src/                     # Source code
│   ├── server/              # MCP server
│   ├── tools/               # Tool implementations
│   ├── metrics/             # Evaluation metrics
│   └── tests/               # Test definitions
└── schemas/                 # JSON schemas
```

## Usage Examples

### Basic Testing

```python
from mcp_client import MCPClient

client = MCPClient("ai-testing")

# Run accuracy tests
result = client.call_tool("run_test_suite", {
    "model": "gpt-4",
    "testCategory": "accuracy",
    "testCases": [
        {
            "input": "What is 2+2?",
            "expected": "4"
        }
    ]
})

print(f"Accuracy: {result.metrics.accuracy}")
```

### Custom Metrics

```python
# Evaluate with custom metrics
evaluation = client.call_tool("evaluate_output", {
    "output": model_response,
    "expected": expected_response,
    "metrics": [
        "exact_match",
        "semantic_similarity",
        "coherence",
        "relevance"
    ]
})
```

## Documentation

- [MCP Protocol](docs/mcp-protocol.md) - Protocol specification
- [Testing Guide](docs/testing-guide.md) - How to write tests
- [Metrics Reference](docs/metrics.md) - Available evaluation metrics
- [API Documentation](docs/api.md) - Complete API reference

## Test Categories

### 1. Accuracy Tests
- Exact match
- Semantic similarity
- Factual correctness
- Mathematical accuracy

### 2. Performance Tests
- Response time
- Throughput
- Token efficiency
- Resource usage

### 3. Security Tests
- Prompt injection
- Jailbreak attempts
- Toxic content
- Bias detection

### 4. Quality Tests
- Coherence
- Fluency
- Relevance
- Completeness

## Configuration

```javascript
module.exports = {
  // Models to test
  models: [
    {
      name: "gpt-4",
      provider: "openai",
      apikey: process.env.OPENAI_API_KEY
    },
    {
      name: "claude-3-opus",
      provider: "anthropic",
      apikey: process.env.ANTHROPIC_API_KEY
    }
  ],

  // Test configurations
  tests: {
    accuracy: {
      enabled: true,
      threshold: 0.95
    },
    performance: {
      maxLatency: 2000,
      maxTokens: 1000
    },
    security: {
      enabled: true,
      strict: true
    }
  },

  // Output format
  reports: {
    format: "json",
    destination: "./test-results"
  }
};
```

## Running Tests

```bash
# Run all tests
npm test

# Run specific category
npm test -- --category accuracy

# Run with specific model
npm test -- --model gpt-4

# Generate report
npm run report
```

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

This project is licensed under the MIT License - see [LICENSE](LICENSE) for details.

## Code of Conduct

Please read [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) to understand our community standards.

## Support

- GitHub Issues: Bug reports and feature requests
- Discussions: Community questions
- MCP Documentation: https://modelcontextprotocol.io

## Related Projects

- [MCP SDK](https://github.com/modelcontextprotocol/sdk)
- [Claude Code](https://claude.ai/code)
- [AI Testing Benchmarks](https://github.com/example/ai-benchmarks)
