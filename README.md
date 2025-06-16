# Bhindi Agent Development Guide

Build powerful AI agents that integrate seamlessly with [Bhindi.io](https://bhindi.io) 🚀 
Bhindi supports 100+ integrations and is the easiest way to build AI agents.

Check a list of integrations available at [Bhindi Agents Directory](https://directory.bhindi.io/)

## 📖 Complete Documentation

**📚 [Read the Full Development Guide →](./development-guide.md)**

For detailed specifications, advanced examples, and comprehensive API reference, see our complete development guide.

---

## 📚 Table of Contents

- [🚀 Quick Start](#-quick-start)
- [🏗️ Agent Architecture](#️-agent-architecture)
- [📋 Core Requirements](#-core-requirements)
- [🔐 Authentication](#-authentication)
- [🧪 Testing Your Agent](#-testing-your-agent)
- [📖 Next Steps](#-next-steps)

---

## 🚀 Quick Start

### 5-Minute Setup

Get your agent up and running in minutes:

```bash
# 1. Clone the starter template
git clone https://github.com/upsurgeio/bhindi-agent-starter.git my-agent
cd my-agent

# 2. Install dependencies
npm install

# 3. Start development server
npm run dev

# 4. Test your agent
curl -X 'POST' \
  'http://localhost:3000/tools/countCharacter' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "character": "s",
  "text": "strawberrry"
}'
```

### Minimal Working Example

Here's a complete, runnable agent:

```typescript
import { Router } from 'express';

const router = Router();

// GET /tools - Return available tools
router.get('/tools', (req, res) => {
  res.json({
    tools: [
      {
        name: 'greet',
        description: 'Send a friendly greeting',
        parameters: {
          type: 'object',
          properties: {
            name: {
              type: 'string',
              description: 'Name of the person to greet'
            }
          },
          required: ['name']
        }
      }
    ]
  });
});

// POST /tools/:toolName - Execute a tool
router.post('/tools/:toolName', (req, res) => {
  const { toolName } = req.params;
  const { name } = req.body;
  
  if (toolName === 'greet') {
    res.json({
      success: true,
      responseType: 'text',
      data: {
        text: `Hello ${name}! Welcome to Bhindi agents!`
      }
    });
  } else {
    res.status(404).json({ 
      success: false,
      error: { message: 'Tool not found', code: 404 }
    });
  }
});

export default router;
```

---

## 🏗️ Agent Architecture

### How Agents Work

[![](https://mermaid.ink/img/pako:eNqNVEtu2zAUvArBVYvKv8iyZKIw0CZpV02KOl208IaRniUiEqk-krEdw4fopttkEaDX6xFKiVY_qWFUC0qPmjejGZLa0lRlQBnV8MWCTOFM8Bx5tZDEXTVHI1JRc2nIRw1IuCY_7r8-tsW_kNeFkJnwoO8PXTnfaAMHCF_l4MYW_PiNfFIW_ZRH-vFCGSDq1ik3ikH7nu07P0AutEFuhJLkTalWvqUB9mYzL84cyvnShhhFrAbCfyt4hIPuWd-eX5GBUarUL69xMHu2EqYg6x6vRe8GNqQAngE-971ty18qxqIk_JaLkl-XQFqeJzrNlzEyL9TqMPCY5SsHI-drSO1xux7iaUnrwEXOKzCA-rDt95fzzveANbcLB_8jAW7d4M1r8sLT6ScxdFSoUtCaoM_8aFTapg12AIgKXYeuldRwMLEzoeuSbxqQLc1_hHVZNyHx0mlpt61SIKdKGlibYwngHuut52Ca_YIk9Z3H130Psk5SyKXCqt2UNKA5iowygxYCWoGbb0q6bcgW1BRQwYIy95hxvFnQhdy5Hnc4PitVdW2obF5QtuSldpWtM266Q_prFkG65TlVVhrKRsm4JaFsS9eURdFJfxjFYRKNwuEkDsOAbiiLx_0ojMJRMp2MT-JoHO4CeteqDvvxNE7iKJ6MpkkUx0kSUMiEUfjO_ymc2aXI6e4n29xrDQ?type=png)](https://mermaid.live/edit#pako:eNqNVEtu2zAUvArBVYvKv8iyZKIw0CZpV02KOl208IaRniUiEqk-krEdw4fopttkEaDX6xFKiVY_qWFUC0qPmjejGZLa0lRlQBnV8MWCTOFM8Bx5tZDEXTVHI1JRc2nIRw1IuCY_7r8-tsW_kNeFkJnwoO8PXTnfaAMHCF_l4MYW_PiNfFIW_ZRH-vFCGSDq1ik3ikH7nu07P0AutEFuhJLkTalWvqUB9mYzL84cyvnShhhFrAbCfyt4hIPuWd-eX5GBUarUL69xMHu2EqYg6x6vRe8GNqQAngE-971ty18qxqIk_JaLkl-XQFqeJzrNlzEyL9TqMPCY5SsHI-drSO1xux7iaUnrwEXOKzCA-rDt95fzzveANbcLB_8jAW7d4M1r8sLT6ScxdFSoUtCaoM_8aFTapg12AIgKXYeuldRwMLEzoeuSbxqQLc1_hHVZNyHx0mlpt61SIKdKGlibYwngHuut52Ca_YIk9Z3H130Psk5SyKXCqt2UNKA5iowygxYCWoGbb0q6bcgW1BRQwYIy95hxvFnQhdy5Hnc4PitVdW2obF5QtuSldpWtM266Q_prFkG65TlVVhrKRsm4JaFsS9eURdFJfxjFYRKNwuEkDsOAbiiLx_0ojMJRMp2MT-JoHO4CeteqDvvxNE7iKJ6MpkkUx0kSUMiEUfjO_ymc2aXI6e4n29xrDQ)

### Authentication & Endpoint Overview

[![](https://mermaid.ink/img/pako:eNqNVMuO0zAU_RXLLGmneTRNxkIjFYgAwahVGbGAsHDjm6nVxA6OPWqm6o4PQII94g_5BJxHy0SaIrLyfZxzro9j73EqGWCCbxUtN-jdKhHIfpVZd4kE__754xuaG70BoXlKNZcC3dQlVAnueptv7n5K8Hz5Br2FGi1EXj9bq8nVbkxLPt5CTVAFqQKd4M8PMJ7FLBpm9BT12MdhbbZplIrftxMQ9ByoAoW03IIY0vqW9gNVnK5zQK-BMlDVP3h3Y7Y2ihOUSiEgbdj7fGoqLQuC7mhu4KQBgnWLR5z6_gut4IvhChiKBSslF3rgU9z49Cq-QRMtZd6NtQJtlKhQk0EMMi54M0Q12FXcmLVcvD8iJ0TQAlp8vIPUaKhQVULKM562TP8371e0KBsxmp-Z1z_JKqikUWmnuVTyjjOraSp7CNY5DTt9TnLuovH4yu69D71h6A_CuG_2-rBv9gd70LU9Wkub8TwnT7Is88EZlLy-BFEWQDQo-UeUD0EWPCzFR0LwMy9jg9J5wtj_O0YELh7Zq8QZJloZGOECVEGbEO8bUILtPSrsz0TsklG1bcw-WExJxUcpiyNMSXO7wSSjeWUjUzKq4SWn9uSKU1ZZm0G9kEZoTLwwbEkw2eOdDQP_wgnCaTBzvNB3Z140wjUm4_BiOpvOwsgNHC9yImd6GOH7Vte5CC_DKAzCmXsZBWEYWQQwrqW67h6I9p04_AFMwEpG?type=png)](https://mermaid.live/edit#pako:eNqNVMuO0zAU_RXLLGmneTRNxkIjFYgAwahVGbGAsHDjm6nVxA6OPWqm6o4PQII94g_5BJxHy0SaIrLyfZxzro9j73EqGWCCbxUtN-jdKhHIfpVZd4kE__754xuaG70BoXlKNZcC3dQlVAnueptv7n5K8Hz5Br2FGi1EXj9bq8nVbkxLPt5CTVAFqQKd4M8PMJ7FLBpm9BT12MdhbbZplIrftxMQ9ByoAoW03IIY0vqW9gNVnK5zQK-BMlDVP3h3Y7Y2ihOUSiEgbdj7fGoqLQuC7mhu4KQBgnWLR5z6_gut4IvhChiKBSslF3rgU9z49Cq-QRMtZd6NtQJtlKhQk0EMMi54M0Q12FXcmLVcvD8iJ0TQAlp8vIPUaKhQVULKM562TP8371e0KBsxmp-Z1z_JKqikUWmnuVTyjjOraSp7CNY5DTt9TnLuovH4yu69D71h6A_CuG_2-rBv9gd70LU9Wkub8TwnT7Is88EZlLy-BFEWQDQo-UeUD0EWPCzFR0LwMy9jg9J5wtj_O0YELh7Zq8QZJloZGOECVEGbEO8bUILtPSrsz0TsklG1bcw-WExJxUcpiyNMSXO7wSSjeWUjUzKq4SWn9uSKU1ZZm0G9kEZoTLwwbEkw2eOdDQP_wgnCaTBzvNB3Z140wjUm4_BiOpvOwsgNHC9yImd6GOH7Vte5CC_DKAzCmXsZBWEYWQQwrqW67h6I9p04_AFMwEpG)

---

## 📋 Core Requirements

Every agent must implement **2 required endpoints** and **1 optional endpoint**:

### Required Endpoints

1. **`GET /tools`** - Returns available tools and their parameters
2. **`POST /tools/:toolName`** - Executes a specific tool

### Optional Endpoints

3. **`POST /resource`** - Provides user/environment context

### Tool Structure

Each tool follows this format:

```typescript
{
  name: string;                    // Tool identifier
  description: string;             // What the tool does
  parameters: {                    // JSON Schema for parameters
    type: 'object';
    properties: Record<string, any>;
    required?: string[];
  };
  confirmationRequired?: boolean;  // Requires user confirmation
  credits?: number;               // Cost in credits
}
```

### Response Format

**Success Response:**
```json
{
  "success": true,
  "responseType": "text|html|media|mixed",
  "data": { "text": "Your response content" }
}
```

**Error Response:**
```json
{
  "success": false,
  "error": {
    "message": "Error description",
    "code": 400,
    "details": "Additional details"
  }
}
```

---

## 🔐 Authentication

All requests require authentication headers:

### Basic Authentication
```bash
x-api-key: your-secret-key
```

### OAuth Authentication (Optional)
```bash
x-api-key: your-secret-key
Authorization: Bearer oauth-token
```

### Variable Headers (Optional)
```bash
x-api-key: your-secret-key
x-dburi: mongodb://connection-string
x-custom-var: custom-value
```

---

## 🧪 Testing Your Agent

### Quick Test Commands

```bash
# Test tools endpoint
curl -H "x-api-key: your-api-key" http://localhost:3000/tools

# Test tool execution
curl -X POST \
  -H "x-api-key: your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"name": "World"}' \
  http://localhost:3000/tools/greet
```

For comprehensive testing examples including OAuth and variable headers, see the [development guide](./development-guide.md#testing-your-agent).

---

## 📖 Next Steps

1. **📚 [Read the Complete Guide](./development-guide.md)** - Detailed specifications and advanced examples
2. **🚀 [Clone the Starter Kit](https://github.com/upsurgeio/bhindi-agent-starter)** - Get started immediately
3. **💬 Join the Community** - [Discord](https://discord.gg/tuXzyx5U) • [Twitter/X](https://x.com/bhindiai)
4. **🆘 Get Support** - [Documentation](https://github.com/upsurgeio/bhindi-docs) • [Email](mailto:info@bhindi.io)
5. **🌐 Visit** - [Bhindi.io](https://bhindi.io)

---

<div align="center">
  <strong>Built with ❤️ for the Bhindi.io ecosystem</strong>
</div>
