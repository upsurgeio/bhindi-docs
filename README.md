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
- [🚀 Bringing Your Custom Agent to Bhindi](#-bringing-your-custom-agent-to-bhindi)
- [📖 Next Steps](#-next-steps)

---

## 🚀 Quick Start

### 5-Minute Setup

Clone the agent-starter-kit repository and get your agent up and running in minutes:
https://github.com/upsurgeio/bhindi-agent-starter

```bash
# 1. Clone the starter template
git clone https://github.com/upsurgeio/bhindi-agent-starter.git starter-kit
cd starter-kit

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

[![](https://mermaid.ink/img/pako:eNp9k82O0zAUhV_F8gpE2sZp02ktVAmYgRUDosMC1I0nuSTWJHa4tqftVH0INmxhMRKvxyPgxA0_Q9UsnNz4O-fmXMU7mukcKKcGPjtQGZxLUaCoV4r4qxFoZSYboSx5bwCJMOTnty_3XfE_8ryUKpcB-vG9L5dbY-GI4bMC_NrB91_JB-0wvApkWC-1BaJvfee2Y9Tt84PyHRTSWBRWakVeVnodJC04WCxCc-4pn8tYYjVxBoj40yEQHj24vrq4IiOrdWWeXuNo8WgtbUk2A9HIwQ1sSQkiB3wctJ3kny7WoSLiVshKXFdAOp8Hfdov42RZ6vVx8FTkK4-Riw1k7nTcgARb0iXwIxc1WEBzPPbbN8s-94i3t0uP_zUB4fwSwhvyJNiZB2PorVBnYAzBMPOTozIua9kRIGr0CtNoZeDoxM6laSqxbSFX2UMMGtECZU65RQcRrQFr0ZZ01-6vqC2hhhXl_jEXeLOiK7X3Gv_nfdS67mWoXVFS_klUxleuyYXtT8DvtwjKZ3-hnbKUp0nnQfmObihPUjaM0yRh4_k4Hc_Z1O9uKR-wZDhhyWQ2nbCYnc3jabKP6F3XNx7O2CSN42TGztLpmM0iCrm0Gl-Hc9gdx_0vCtcxng?type=png)](https://mermaid.live/edit#pako:eNp9k82O0zAUhV_F8gpE2sZp02ktVAmYgRUDosMC1I0nuSTWJHa4tqftVH0INmxhMRKvxyPgxA0_Q9UsnNz4O-fmXMU7mukcKKcGPjtQGZxLUaCoV4r4qxFoZSYboSx5bwCJMOTnty_3XfE_8ryUKpcB-vG9L5dbY-GI4bMC_NrB91_JB-0wvApkWC-1BaJvfee2Y9Tt84PyHRTSWBRWakVeVnodJC04WCxCc-4pn8tYYjVxBoj40yEQHj24vrq4IiOrdWWeXuNo8WgtbUk2A9HIwQ1sSQkiB3wctJ3kny7WoSLiVshKXFdAOp8Hfdov42RZ6vVx8FTkK4-Riw1k7nTcgARb0iXwIxc1WEBzPPbbN8s-94i3t0uP_zUB4fwSwhvyJNiZB2PorVBnYAzBMPOTozIua9kRIGr0CtNoZeDoxM6laSqxbSFX2UMMGtECZU65RQcRrQFr0ZZ01-6vqC2hhhXl_jEXeLOiK7X3Gv_nfdS67mWoXVFS_klUxleuyYXtT8DvtwjKZ3-hnbKUp0nnQfmObihPUjaM0yRh4_k4Hc_Z1O9uKR-wZDhhyWQ2nbCYnc3jabKP6F3XNx7O2CSN42TGztLpmM0iCrm0Gl-Hc9gdx_0vCtcxng)

### Authentication & Endpoint Overview

[![](https://mermaid.ink/img/pako:eNqNVMuO0zAU_RXLLGmnSdxHaqGRCkSAYNSqjFhAWLjJzdSaxA5-jJqpuuMDkGCP-EM-AefRMpGmiKx8H-ec6-PYe5zIFDDFN4qVW_RuHQvkPm03bSLGv3_--IYW1mxBGJ4ww6VA11UJOsZtb_0t_E8xXqzeoLdQoaXIq2cbNbrcDVnJh7dQUaQhUWBi_PkBJnCYZc2MnqIO-zisydaNUvH7ZgKKngNToJCRtyD6tMTRfmCKs00O6DWwFJT-B-9umG6s4hQlUghIavYun1htZEHRHcstnDRApO3iEae-_0Jr-GK5ghRFIi0lF6bnU1T79Cq6RiMjZd6OtQZjldCozqAUMi54PYTu7SqqzVot3x-RIypYAQ0-2kFiDWikS0h4xpOG6f_m_YqWZS3G8jPzkpOsAi2tSlrNlZJ3PHWaVrtDcM4Z2JlzkgsfDYeXbu9dGPRD0gujrjnowq6Z9PZgKne0jjbjeU6fZFlGwOuVgq4EYTaBsFciRxSBSTZ5WIqOhECyIEt7pfOEEfk7Rgg-HrirxFNMjbIwwAWogtUh3tegGLt7VLifibrlhmmozT44TMnERymLI0xJe7PFNGO5dpEtU2bgJWfu5IpTVjmbQb2QVhhMg3HYkGC6xztM_WByMRsT4s1n0znxZtPxAFeua3JBvNAl534YTsbzYHoY4PtG17sIpyEJpx7x_PnMH5PpAEPKjVRX7QPRvBOHP0ocSjY?type=png)](https://mermaid.live/edit#pako:eNqNVMuO0zAU_RXLLGmnSdxHaqGRCkSAYNSqjFhAWLjJzdSaxA5-jJqpuuMDkGCP-EM-AefRMpGmiKx8H-ec6-PYe5zIFDDFN4qVW_RuHQvkPm03bSLGv3_--IYW1mxBGJ4ww6VA11UJOsZtb_0t_E8xXqzeoLdQoaXIq2cbNbrcDVnJh7dQUaQhUWBi_PkBJnCYZc2MnqIO-zisydaNUvH7ZgKKngNToJCRtyD6tMTRfmCKs00O6DWwFJT-B-9umG6s4hQlUghIavYun1htZEHRHcstnDRApO3iEae-_0Jr-GK5ghRFIi0lF6bnU1T79Cq6RiMjZd6OtQZjldCozqAUMi54PYTu7SqqzVot3x-RIypYAQ0-2kFiDWikS0h4xpOG6f_m_YqWZS3G8jPzkpOsAi2tSlrNlZJ3PHWaVrtDcM4Z2JlzkgsfDYeXbu9dGPRD0gujrjnowq6Z9PZgKne0jjbjeU6fZFlGwOuVgq4EYTaBsFciRxSBSTZ5WIqOhECyIEt7pfOEEfk7Rgg-HrirxFNMjbIwwAWogtUh3tegGLt7VLifibrlhmmozT44TMnERymLI0xJe7PFNGO5dpEtU2bgJWfu5IpTVjmbQb2QVhhMg3HYkGC6xztM_WByMRsT4s1n0znxZtPxAFeua3JBvNAl534YTsbzYHoY4PtG17sIpyEJpx7x_PnMH5PpAEPKjVRX7QPRvBOHP0ocSjY)

---

## 📋 Core Requirements

Every agent must implement **2 required endpoints** and **1 optional endpoint**:

### Required Endpoints

1. **`GET /tools`** - Returns available tools and their parameters
2. **`POST /tools/:toolName`** - Executes a specific tool

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

To let agents use integrations like GitHub, Gmail, Discord etc from the [Bhindi Agents Directory](https://directory.bhindi.io/), requests to the agent endpoint include the below headers.

Read [Connecting your agent to Bhindi](#-bringing-your-custom-agent-to-bhindi) for more details.

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

## 🚀 Bringing Your Custom Agent to Bhindi

Once you've built and deployed your agent, you can easily integrate it with [Bhindi.io](https://bhindi.io) to make it accessible through the platform.

### Integration Process

1. **Create & Deploy Your Agent**
   - Build your agent using this development guide
   - Deploy it to a publicly accessible endpoint
   - Ensure both `/tools` and `/tools/:toolName` endpoints are working

2. **Add Agent to Your Bhindi Account**
   - Start a new chat on [Bhindi.io](https://bhindi.io)
   - Use the following message format:

```
Add my agent using Bhindi Agent Manager. The details are as follows:

id: my-special-calculator
name: Special Calculator
description: A powerful calculator that can perform complex mathematical operations and generate visualizations
endpoint: https://my-calculator-app-here.com
```

3. **With OAuth Integration (Optional)**
   - For agents that need access to services like GitHub, Google, Discord, etc.
   - First connect the required apps at [bhindi.io/apps](https://bhindi.io/apps)
   - Include OAuth services in your agent registration:

```
Add my agent using Bhindi Agent Manager. The details are as follows:

id: my-github-analyzer
name: GitHub Repository Analyzer
description: Analyzes GitHub repositories and provides insights
endpoint: https://my-github-analyzer.com
oauth: github, discord
```

### Required Information

- **id**: Unique identifier for your agent (lowercase, hyphens allowed)
- **name**: Display name for your agent
- **description**: What your agent does and its capabilities
- **endpoint**: Public HTTPS URL where your agent is deployed
- **oauth** (optional): Comma-separated list of services your agent needs access to

### OAuth Access

For agents requiring OAuth access:

1. **Connect Apps First**: Visit [bhindi.io/apps](https://bhindi.io/apps) and connect the required services (GitHub, Google, Discord, etc.)
2. **Account Access**: Your custom agent will automatically get authentication access for the connected accounts
3. **Supported Services**: Include any of the 100+ integrations available in the Bhindi ecosystem

### Integration Support

The **Bhindi Agent Manager** will guide you through the integration process and help resolve any issues during setup.

---

## 📖 Next Steps

1. **📚 [Detailed specifications and advanced examples](./development-guide.md)**
2. **🚀 [Clone the Starter Kit](https://github.com/upsurgeio/bhindi-agent-starter)** - Get started immediately
3. **💬 Join the Community** - [Discord](https://discord.gg/hSfTG33ymy) • [Twitter/X](https://x.com/bhindiai)
4. **🆘 Get Support** - [Documentation](https://github.com/upsurgeio/bhindi-docs) • [Email](mailto:info@bhindi.io)
5. **🌐 Visit** - [Bhindi.io](https://bhindi.io)

---

<div align="center">
  <strong>Built with ❤️ for the Bhindi.io ecosystem</strong>
</div>
