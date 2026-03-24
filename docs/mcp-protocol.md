# Model Context Protocol (MCP) Guide

This guide explains how the AI Testing MCP server implements and uses the Model Context Protocol for AI testing workflows.

## What is MCP?

The Model Context Protocol (MCP) is an open standard that enables AI assistants to interact with external tools, data sources, and systems through a standardized interface.

## MCP Core Concepts

### 1. Transport Layer

MCP operates over JSON-RPC 2.0, supporting multiple transports:

```typescript
// STDIO Transport (most common)
const transport = new StdioClientTransport({
  command: "node",
  args: ["path/to/mcp-server"]
});

// SSE Transport (for web)
const transport = new SSEClientTransport(
  "http://localhost:3000/sse"
);
```

### 2. Server Initialization

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server(
  {
    name: "ai-testing-mcp",
    version: "1.0.0"
  },
  {
    capabilities: {
      tools: {},
      resources: {},
      prompts: {}
    }
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

## Server Capabilities

### Tools (Callable Functions)

Tools are the primary way clients interact with the MCP server.

```typescript
// Define a tool
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case "run_test_suite":
      return await runTestSuite(args);

    case "evaluate_output":
      return await evaluateOutput(args);

    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});
```

### Tools Registration

```typescript
// List available tools
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: "run_test_suite",
        description: "Run tests on an AI model",
        inputSchema: {
          type: "object",
          properties: {
            model: { type: "string" },
            testCategory: {
              type: "string",
              enum: ["accuracy", "performance", "security", "quality"]
            },
            testCases: {
              type: "array",
              items: {
                type: "object",
                properties: {
                  input: { type: "string" },
                  expected: { type: "string" }
                },
                required: ["input", "expected"]
              }
            }
          },
          required: ["model", "testCategory"]
        }
      }
    ]
  };
});
```

### Resources (Data Access)

Resources provide access to data and files.

```typescript
// List resources
server.setRequestHandler(ListResourcesRequestSchema, async () => {
  return {
    resources: [
      {
        uri: "test://data/accuracy",
        name: "Accuracy Test Data",
        description: "Sample data for accuracy tests",
        mimeType: "application/json"
      },
      {
        uri: "test://config/default",
        name: "Default Configuration",
        description: "Default test configuration",
        mimeType: "application/json"
      }
    ]
  };
});

// Read resource
server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  const uri = request.params.uri;

  if (uri === "test://data/accuracy") {
    return {
      contents: [
        {
          uri,
          mimeType: "application/json",
          text: JSON.stringify({
            test_cases: [
              { input: "2+2", expected: "4" },
              { input: "What is the capital of France?", expected: "Paris" }
            ]
          })
        }
      ]
    };
  }

  throw new Error(`Resource not found: ${uri}`);
});
```

### Prompts (Reusable Templates)

Prompts provide reusable prompt templates.

```typescript
// List prompts
server.setRequestHandler(ListPromptsRequestSchema, async () => {
  return {
    prompts: [
      {
        name: "test_generation",
        description: "Generate test cases for a feature",
        arguments: [
          {
            name: "feature",
            description: "Feature to test",
            required: true
          },
          {
            name: "count",
            description: "Number of test cases",
            required: false
          }
        ]
      }
    ]
  };
});

// Get prompt
server.setRequestHandler(GetPromptRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "test_generation") {
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `Generate ${args.count || 5} test cases for ${args.feature}.\n\n` +
                  `Each test case should include:\n` +
                  `- Test scenario\n` +
                  `- Input\n` +
                  `- Expected output\n` +
                  `- Edge cases to consider`
          }
        }
      ]
    };
  }

  throw new Error(`Prompt not found: ${name}`);
});
```

## Client Implementation

### Connecting to MCP Server

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";

// Create client
const client = new Client(
  {
    name: "ai-testing-client",
    version: "1.0.0"
  },
  {
    capabilities: {}
  }
);

// Connect to server
const transport = new StdioClientTransport({
  command: "node",
  args: ["dist/index.js"]
});

await client.connect(transport);
```

### Calling Tools

```typescript
// Call a tool
const result = await client.callTool({
  name: "run_test_suite",
  arguments: {
    model: "gpt-4",
    testCategory: "accuracy",
    testCases: [
      {
        input: "What is 2+2?",
        expected: "4"
      }
    ]
  }
});

console.log(result.content);
```

### Reading Resources

```typescript
// Read a resource
const resource = await client.readResource({
  uri: "test://data/accuracy"
});

console.log(resource.contents[0].text);
```

## Tool Implementation Examples

### Test Suite Runner

```typescript
async function runTestSuite(args: any): Promise<ToolResponse> {
  const { model, testCategory, testCases } = args;

  // Initialize model client
  const modelClient = getModelClient(model);

  // Run tests
  const results = [];
  for (const testCase of testCases) {
    const response = await modelClient.complete(testCase.input);
    const passed = evaluate(response, testCase.expected);

    results.push({
      input: testCase.input,
      expected: testCase.expected,
      actual: response,
      passed
    });
  }

  // Calculate metrics
  const accuracy = results.filter(r => r.passed).length / results.length;

  return {
    content: [
      {
        type: "text",
        text: JSON.stringify({
          category: testCategory,
          model,
          total_tests: testCases.length,
          passed: results.filter(r => r.passed).length,
          accuracy,
          results
        }, null, 2)
      }
    ]
  };
}
```

### Output Evaluator

```typescript
async function evaluateOutput(args: any): Promise<ToolResponse> {
  const { output, expected, metrics } = args;

  const evaluationResults = {};

  for (const metric of metrics) {
    switch (metric) {
      case "exact_match":
        evaluationResults.exact_match = output === expected;
        break;

      case "semantic_similarity":
        evaluationResults.semantic_similarity =
          await calculateSimilarity(output, expected);
        break;

      case "coherence":
        evaluationResults.coherence =
          await evaluateCoherence(output);
        break;

      case "relevance":
        evaluationResults.relevance =
          await evaluateRelevance(output, expected);
        break;
    }
  }

  return {
    content: [
      {
        type: "text",
        text: JSON.stringify(evaluationResults, null, 2)
      }
    ]
  };
}
```

### Test Case Generator

```typescript
async function generateTestCases(args: any): Promise<ToolResponse> {
  const { scenario, count = 5 } = args;

  // Use LLM to generate test cases
  const llmClient = new LLMClient(process.env.OPENAI_API_KEY);

  const prompt = `Generate ${count} test cases for: ${scenario}

For each test case, provide:
- Test name
- Description
- Input
- Expected output
- Edge cases to consider`;

  const response = await llmClient.complete(prompt);

  return {
    content: [
      {
        type: "text",
        text: response
      }
    ]
  };
}
```

## Error Handling

### Standard Error Response

```typescript
try {
  const result = await someOperation();
  return {
    content: [
      {
        type: "text",
        text: JSON.stringify(result, null, 2)
      }
    ]
  };
} catch (error) {
  return {
    content: [
      {
        type: "text",
        text: JSON.stringify({
          error: error.message,
          code: error.code || "UNKNOWN_ERROR"
        }, null, 2)
      }
    ],
    isError: true
  };
}
```

### Custom Error Types

```typescript
class MCPError extends Error {
  constructor(
    message: string,
    public code: string,
    public details?: any
  ) {
    super(message);
    this.name = "MCPError";
  }
}

// Usage
throw new MCPError(
  "Model not supported",
  "UNSUPPORTED_MODEL",
  { model: "unknown-model", supported: ["gpt-4", "claude-3"] }
);
```

## Advanced Features

### Streaming Responses

```typescript
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const stream = new ReadableStream({
    async start(controller) {
      for (let i = 0; i < 10; i++) {
        controller.enqueue({
          type: "text",
          text: `Chunk ${i}\n`
        });
        await sleep(100);
      }
      controller.close();
    }
  });

  return {
    content: [
      {
        type: "resource",
        uri: "stream://output",
        mimeType: "text/plain"
      }
    ]
  };
});
```

### Progress Updates

```typescript
interface ProgressUpdate {
  progress: number;
  total: number;
  message: string;
}

async function runLongRunningTask(): Promise<ToolResponse> {
  const total = 100;
  const results = [];

  for (let i = 0; i < total; i++) {
    // Process item
    results.push(await processItem(i));

    // Send progress update
    // (This would be handled by your notification system)
    const progress: ProgressUpdate = {
      progress: i + 1,
      total,
      message: `Processed ${i + 1}/${total} items`
    };

    // Emit progress event
    // server.notification("progress", progress);
  }

  return {
    content: [
      {
        type: "text",
        text: JSON.stringify({ results, total }, null, 2)
      }
    ]
  };
}
```

## Testing MCP Tools

### Unit Testing

```typescript
import { describe, it, expect } from "@jest/globals";

describe("runTestSuite", () => {
  it("should run test suite successfully", async () => {
    const result = await runTestSuite({
      model: "gpt-4",
      testCategory: "accuracy",
      testCases: [
        { input: "2+2", expected: "4" }
      ]
    });

    expect(result.content).toBeDefined();
    expect(result.content[0].type).toBe("text");
  });
});
```

### Integration Testing

```typescript
describe("MCP Server Integration", () => {
  let client: Client;
  let serverProcess: ChildProcess;

  beforeAll(async () => {
    // Start server
    serverProcess = spawn("node", ["dist/index.js"]);

    // Connect client
    client = new Client({
      name: "test-client",
      version: "1.0.0"
    }, {});

    await client.connect(new StdioClientTransport({
      command: "node",
      args: ["dist/index.js"]
    }));
  });

  it("should list available tools", async () => {
    const tools = await client.listTools();
    expect(tools.tools).toHaveLength(3);
  });

  afterAll(async () => {
    await client.close();
    serverProcess.kill();
  });
});
```

## Configuration

### Server Configuration

```typescript
interface MCPServerConfig {
  name: string;
  version: string;
  capabilities: {
    tools?: boolean;
    resources?: boolean;
    prompts?: boolean;
  };
  transport: {
    type: "stdio" | "sse";
    options: any;
  };
}

const config: MCPServerConfig = {
  name: "ai-testing-mcp",
  version: "1.0.0",
  capabilities: {
    tools: true,
    resources: true,
    prompts: true
  },
  transport: {
    type: "stdio",
    options: {}
  }
};
```

### Client Configuration

```typescript
interface MCPClientConfig {
  serverPath: string;
  timeout: number;
  retries: number;
  logLevel: "debug" | "info" | "error";
}

const config: MCPClientConfig = {
  serverPath: "./dist/index.js",
  timeout: 30000,
  retries: 3,
  logLevel: "info"
};
```

## Best Practices

### 1. Input Validation

Always validate input parameters:

```typescript
function validateTestSuiteArgs(args: any): void {
  if (!args.model) {
    throw new MCPError("Model is required", "INVALID_INPUT");
  }

  if (!["accuracy", "performance", "security", "quality"].includes(args.testCategory)) {
    throw new MCPError("Invalid test category", "INVALID_INPUT");
  }

  if (!Array.isArray(args.testCases)) {
    throw new MCPError("Test cases must be an array", "INVALID_INPUT");
  }
}
```

### 2. Rate Limiting

Implement rate limiting for expensive operations:

```typescript
const rateLimiter = new RateLimiter({
  tokensPerInterval: 10,
  interval: "second"
});

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  await rateLimiter.removeTokens(1);
  // Process request
});
```

### 3. Logging

Use structured logging:

```typescript
import pino from "pino";

const logger = pino({
  level: process.env.LOG_LEVEL || "info"
});

logger.info({
  tool: "run_test_suite",
  model: args.model,
  testCount: args.testCases.length
}, "Running test suite");
```

## Resources

- [MCP Specification](https://modelcontextprotocol.io/specification/)
- [MCP SDK TypeScript](https://github.com/modelcontextprotocol/typescript-sdk)
- [MCP Inspector](https://github.com/modelcontextprotocol/inspector)
- [Example MCP Servers](https://github.com/modelcontextprotocol/servers)
