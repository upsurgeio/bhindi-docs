# Agent Development Guide

This guide explains how to create external agents that integrate with the system. Each agent must implement specific endpoints and follow authentication patterns.

## Required Structure

Every agent must implement **2 required endpoints (GET /tools and POST /tools/:toolName)**:

### 1. GET /tools (Required)

Returns available tools for the agent.

**Response Type:** `ToolsResponseDto`

```typescript
export class ToolsResponseDto {
  tools: ToolDto[];
}

export class ToolDto {
  name: string; // Tool identifier
  description: string; // What the tool does
  parameters: ToolParameterDto; // JSON Schema for parameters
  confirmationRequired?: boolean; // If true, the tool will require user confirmation
  credits?: number; // Cost in credits (optional: important for billing costly functions like Image generation, Video generation, etc.)
}

export class ToolParameterDto {
  type: 'object';
  properties: Record<string, PropertyDto>; // List of properties for the object
  required?: string[]; // List of required parameters
}

export class PropertyDto {
  type: 'string' | 'number' | 'boolean' | 'array' | 'object';
  description: string;
  enum?: string[]; // If the parameter is an enum, list the possible values
  default?: string | number | boolean; // Default value for the parameter
  items?: PropertyDto; // If the parameter is an array, list the items
}
```

**Example from Slack Agent:**

```typescript
tools: [
  {
    name: 'sendMessage',
    description:
      'Send a message to a Slack user or channel. No need for exact name, it searches through all users and channels. Can directly use this to send message to a user or channel, no need to get the id first if user or channel name is known.',
    parameters: {
      type: 'object',
      properties: {
        query: {
          type: 'string',
          description:
            'Name of the user/channel to send message to. Can be a user name, channel name, or a channel id. Never pass a user id, always pass a user name or channel name.',
        },
        type: {
          type: 'string',
          description:
            'Type of recipient: "user", "channel", "id", or "unknown".',
          enum: ['user', 'channel', 'id', 'unknown'],
          default: 'unknown',
        },
        text: {
          type: 'string',
          description: 'Text content of the message',
        },
        threadTs: {
          type: 'string',
          description: 'Optional thread timestamp to reply to a thread',
        },
      },
      required: ['query', 'text'],
    },
    confirmationRequired: true,
  },
];
```

### 2. POST /tools/:toolName (Required)

Executes a specific tool with provided parameters.

**Parameters:**

- `toolName` (path parameter): Name of the tool to execute
- Request body: Tool-specific parameters

**Response Type:** `BaseErrorResponseDto | BaseSuccessResponseDto<any>`

```typescript
export class BaseErrorResponseDto {
  success: false;
  error: {
    message: string;
    code: number | string;
    details: string;
  };

  constructor(
    message: string,
    code: number | string = 500,
    details: string = '',
  ) {
    this.success = false;
    this.error = {
      message,
      code,
      details,
    };
  }
}

export class BaseSuccessResponseDto<T> {
  success: true;
  data: {
    [key: string]: T;
  };

  constructor(data: T) {
    this.success = true;
    this.data = data;
  }
}

export class ResponseMediaItem {
  type: string;
  url: string;
  mimeType: string;
  description: string;
  metadata: object;

  constructor(
    type: string,
    url: string,
    mimeType: string,
    description: string,
    metadata: object = {},
  ) {
    this.type = type;
    this.url = url;
    this.mimeType = mimeType;
    this.description = description;
    this.metadata = metadata;
  }
}
```

## Authentication

To let agents use integrations like GitHub, GMail etc from the [Bhindi Agents Directory](https://directory.bhindi.io/), 
requests to agents include the following headers:

### Basic Authentication
```bash
x-api-key: your-secret-key
```

- **Required for all endpoints**
- Used to authenticate the server making the request
- Every agent must validate this header

### OAuth Authentication (Optional)
```bash
x-api-key: your-secret-key
Authorization: Bearer oauth-token
```

- Used for agents requiring user-specific OAuth tokens
- The `Authorization` header contains the OAuth token for services like Twitter, Slack, GitHub, etc.
- The `x-api-key` header is still required for server authentication
- Examples: Twitter agent, Slack agent, GitHub agent

### Variable Headers (Optional)
```bash
x-api-key: your-secret-key
x-dburi: mongodb://connection-string
x-custom-var: custom-value
```

- Used for agent-specific configuration
- Pattern: `x-<variablename>` where `variablename` is your custom variable
- The `x-api-key` header is still required for server authentication
- Examples:
  - `x-dburi`: MongoDB connection string for database agents
  - `x-custom-var`: Any custom configuration your agent needs

### Implementation Notes

- **API Key Validation**: Always validate the `x-api-key` header first
- **OAuth Token Handling**: Extract OAuth tokens from the `Authorization` header when needed
- **Custom Variables**: Parse custom headers starting with `x-` for agent-specific configuration
- **Security**: Never log or expose authentication tokens in responses or logs

## Tool Definition Structure

Each tool in the `ToolsResponseDto` follows this structure:

```typescript
{
  name: string;                    // Tool identifier
  description: string;             // What the tool does
  parameters: {                    // JSON Schema for parameters
    type: 'object';
    properties: Record<string, any>;
    required?: string[];
  };
  visibleParameters?: string[];    // Parameters shown in UI
  confirmationRequired?: boolean;  // Requires user confirmation
  credits?: number;               // Cost in credits
}
```

## Best Practices

1. **Error Handling**: Always return proper error responses using `BaseErrorResponseDto`
2. **Parameter Validation**: Validate all input parameters according to the tool schema
3. **Authentication**: Implement proper guards for API key and OAuth validation
4. **Documentation**: Provide clear descriptions for tools and parameters
5. **Response Types**: Use appropriate response types (text, html, media, mixed)
6. **Confirmation**: Set `confirmationRequired: true` for critical operations
