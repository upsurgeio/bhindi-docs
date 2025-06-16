# Agent Development Guide

This guide explains how to create external agents that integrate with the system. Each agent must implement specific endpoints and follow authentication patterns.

## Required Structure

Every agent must implement **2 required endpoints (GET /tools and POST /tools/:toolName)** and **1 optional endpoint (POST /resource)**:

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
  responseType: string;
  data: {
    [key: string]: T;
  };

  constructor(data: T, responseType: 'text' | 'html' | 'media' | 'mixed') {
    this.success = true;
    this.responseType = responseType;
    if (responseType === 'text') {
      this.data = {
        text: data,
      };
    } else if (responseType === 'html') {
      this.data = {
        html: data,
      };
    } else if (responseType === 'media') {
      this.data = {
        media: data,
      };
    } else if (responseType === 'mixed') {
      // @ts-expect-error - data is of type T
      this.data = {
        ...data,
      };
    }
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

### 3. POST /resource (Optional)

Provides contextual information about the current user or environment to help the agent work effectively.

**Example Use Cases:**

- GitHub agent: Return user's profile and repositories
- MongoDB agent: Return database schema information
- Slack agent: Return user's workspace info and frequent contacts

## Authentication

All requests include authentication headers:

### API Key Authentication

- Header: `x-api-key`
  - Used to authenticate the server making the request
  - Required for all endpoints

### OAuth Authentication (Optional)

- Header: `x-api-key`
  - Used to authenticate the server making the request
  - Required for all endpoints
- Header: `Authorization: Bearer <token>`
  - Used for agents requiring user-specific OAuth tokens
  - Examples: Twitter, Slack, GitHub agents

### Variable Headers (Optional)

- Header: `x-api-key`
  - Used to authenticate the server making the request
  - Required for all endpoints
- Header: `x-<variablename>`
  - Used for agent-specific configuration
  - Example: `x-dburi` for MongoDB agent

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
