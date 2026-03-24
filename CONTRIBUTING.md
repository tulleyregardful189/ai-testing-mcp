# Contributing to AI Testing MCP

Thank you for your interest in contributing to AI Testing MCP! This document provides guidelines and instructions for contributing to the Model Context Protocol server for AI testing.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Contributing Guidelines](#contributing-guidelines)
- [Pull Request Process](#pull-request-process)
- [MCP Protocol Guidelines](#mcp-protocol-guidelines)

## Code of Conduct

Please read and follow our [Code of Conduct](CODE_OF_CONDUCT.md).

## Getting Started

### Prerequisites

- Node.js 18+
- TypeScript 5+
- npm or yarn
- Git
- API keys for:
  - OpenAI (for testing)
  - Anthropic (for testing)

### Setting Up Your Development Environment

1. **Fork and clone the repository**

   ```bash
   git clone https://github.com/groovy-web/ai-testing-mcp.git
   cd ai-testing-mcp
   ```

2. **Install dependencies**

   ```bash
   npm install
   ```

3. **Set up environment**

   ```bash
   cp .env.example .env
   # Edit .env with your test API keys
   ```

4. **Build the project**

   ```bash
   npm run build
   ```

5. **Run tests**

   ```bash
   npm test
   ```

## Development Setup

### MCP Server Architecture

The MCP server is organized into:

- **Tools**: Callable testing tools
- **Resources**: Test data and configurations
- **Prompts**: Reusable test prompt templates
- **Metrics**: Evaluation metric implementations

### Development Workflow

1. **Create a feature branch**

   ```bash
   git checkout -b feature/your-feature
   ```

2. **Implement your feature**

   - Add tools to `src/tools/`
   - Add metrics to `src/metrics/`
   - Update schemas in `schemas/`
   - Write tests in `tests/`

3. **Test locally**

   ```bash
   npm run lint
   npm test
   npm run build
   ```

4. **Test with MCP client**

   ```bash
   # Start the server
   npm start

   # Connect with MCP client (in another terminal)
   mcp-client connect
   ```

## Contributing Guidelines

### What to Contribute

We welcome contributions in:

- **New Testing Tools**: Additional MCP tools for testing
- **Evaluation Metrics**: New metrics for evaluation
- **Test Strategies**: Novel testing approaches
- **Documentation**: Guides and API docs
- **Bug Fixes**: Resolving issues
- **Performance**: Optimization and improvements

### Adding New Tools

Tools implement the MCP tool specification:

```typescript
// src/tools/my-tool.ts
import { Tool } from '@modelcontextprotocol/sdk/types.js';

export const myTool: Tool = {
  name: 'my_testing_tool',
  description: 'Description of what the tool does',
  inputSchema: {
    type: 'object',
    properties: {
      param1: {
        type: 'string',
        description: 'Parameter description'
      }
    },
    required: ['param1']
  }
};

export async function executeMyTool(args: any) {
  // Implementation
  return {
    content: [{
      type: 'text',
      text: 'Result'
    }]
  };
}
```

### Adding New Metrics

Metrics should implement the base metric interface:

```typescript
// src/metrics/my-metric.ts
export interface MetricResult {
  name: string;
  value: number;
  passed: boolean;
  details?: string;
}

export async function calculateMyMetric(
  output: string,
  expected: string
): Promise<MetricResult> {
  // Implementation
  return {
    name: 'my_metric',
    value: 0.95,
    passed: true,
    details: 'Score details'
  };
}
```

### Code Style

Follow TypeScript best practices:

```typescript
// Use strict typing
interface TestConfig {
  model: string;
  timeout: number;
  retries?: number;
}

// Use async/await
async function runTest(config: TestConfig): Promise<TestResult> {
  try {
    const result = await executeTest(config);
    return result;
  } catch (error) {
    logger.error(`Test failed: ${error}`);
    throw error;
  }
}

// Error handling
class TestError extends Error {
  constructor(message: string, public code: string) {
    super(message);
    this.name = 'TestError';
  }
}

// Documentation
/**
 * Runs a test suite on the specified model.
 *
 * @param config - Test configuration
 * @returns Test results with metrics
 * @throws TestError if test execution fails
 */
```

### Testing Standards

All code must include tests:

```typescript
import { describe, it, expect } from '@jest/globals';

describe('MyTool', () => {
  it('should execute successfully', async () => {
    const result = await executeMyTool({ param1: 'test' });
    expect(result.content).toBeDefined();
  });

  it('should handle errors gracefully', async () => {
    await expect(
      executeMyTool({ param1: '' })
    ).rejects.toThrow('Invalid parameter');
  });
});
```

### Commit Message Format

Follow conventional commits:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

**Examples**:

```
feat(tools): add semantic similarity metric

fix(server): resolve tool execution timeout

docs: update MCP protocol guide

test: add integration tests for evaluation

perf: optimize batch test execution
```

## Pull Request Process

### 1. Before Creating PR

- [ ] Code follows style guidelines
- [ ] Tests added and passing (100% coverage for new code)
- [ ] Documentation updated
- [ ] Typescript compiles without errors
- [ ] Linter passes

### 2. Creating PR

1. **Descriptive title**
   ```
   feat(tools): add new evaluation metric
   ```

2. **Detailed description**
   - What changes were made
   - Why changes were needed
   - How the tool/metric works
   - Usage examples

3. **MCP compliance**
   - Tool schema is valid
   - Input/output follows MCP spec
   - Error handling is proper

4. **Testing evidence**
   - Unit tests pass
   - Integration tests pass
   - Manual testing results

### 3. PR Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New tool/metric
- [ ] Documentation update
- [ ] Performance improvement
- [ ] MCP protocol update

## MCP Compliance
- [ ] Follows MCP tool specification
- [ ] Schema is valid
- [ ] Error handling implemented
- [ ] Returns proper MCP responses

## Testing
- [ ] Unit tests added
- [ ] Integration tests added
- [ ] All tests pass
- [ ] Manual testing completed

## Documentation
- [ ] Tool documentation added
- [ ] Examples provided
- [ ] API docs updated
- [ ] README updated

## Checklist
- [ ] Code follows style guide
- [ ] Self-review completed
- [ ] No new warnings
- [ ] Performance tested
```

### Review Process

#### What We Review

1. **MCP Compliance**
   - Tool schema follows specification
   - Input validation is proper
   - Error responses are correct
   - Resource management is sound

2. **Code Quality**
   - TypeScript best practices
   - Error handling
   - Performance considerations
   - Security (no API key leaks)

3. **Testing**
   - Comprehensive test coverage
   - Edge cases covered
   - Error scenarios tested
   - Performance tested

4. **Documentation**
   - Clear tool description
   - Input/output documented
   - Examples provided
   - Usage patterns shown

#### Timeline

- Initial review: 2-3 days
- Additional rounds: 2-3 days each
- Complex features: 1 week

## MCP Protocol Guidelines

### Tool Definition

Tools must follow MCP tool specification:

```typescript
{
  name: string;              // Unique tool identifier
  description: string;       // What the tool does
  inputSchema: JSONSchema;   // Valid JSON Schema
}
```

### Tool Response

Tools must return valid MCP responses:

```typescript
{
  content: Array<{
    type: 'text' | 'image' | 'resource';
    text?: string;
    data?: string;
    uri?: string;
  }>;
  isError?: boolean;
}
```

### Error Handling

```typescript
try {
  // Tool logic
  return {
    content: [{ type: 'text', text: 'Success' }]
  };
} catch (error) {
  return {
    content: [{
      type: 'text',
      text: `Error: ${error.message}`
    }],
    isError: true
  };
}
```

### Resource Management

```typescript
// Define resources
server.setRequestHandler(ListResourcesRequestType, async () => ({
  resources: [
    {
      uri: 'test://data',
      name: 'Test Data',
      description: 'Sample test data',
      mimeType: 'application/json'
    }
  ]
}));
```

## Testing Guidelines

### Local Testing

```bash
# Run unit tests
npm test

# Run with coverage
npm run test:coverage

# Run integration tests
npm run test:integration

# Run linter
npm run lint

# Type check
npm run type-check
```

### Manual Testing with MCP Client

```bash
# Start server
npm start

# In another terminal, test with MCP Inspector
npx @modelcontextprotocol/inspector
```

### Before Submitting

1. **Format code**
   ```bash
   npm run format
   ```

2. **Lint code**
   ```bash
   npm run lint
   ```

3. **Run tests**
   ```bash
   npm test
   ```

4. **Build project**
   ```bash
   npm run build
   ```

## Feature Proposal Process

For new testing tools or metrics:

1. **Open a discussion** with proposal
2. **Get feedback** from maintainers
3. **Create specification** document
4. **Get approval** before implementing

## Release Process

Maintainers handle releases:

1. Update version in package.json
2. Update CHANGELOG.md
3. Create git tag
4. Publish to npm
5. Create GitHub release

## Getting Help

### Resources

- [MCP Documentation](https://modelcontextprotocol.io)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [Testing Guide](docs/testing-guide.md)

### Community

- GitHub Issues: Bug reports, questions
- GitHub Discussions: Ideas, proposals
- MCP Discord: Community chat

### Contact Maintainers

- Create a GitHub issue
- Mention @maintainer in discussions
- Check documentation first

## Recognition

Contributors will be:
- Listed in CONTRIBUTORS.md
- Mentioned in release notes
- Credited in tools/metrics they create

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

## Questions?

- Check existing issues
- Read MCP documentation
- Ask in GitHub Discussions

Thank you for contributing to AI Testing MCP!
