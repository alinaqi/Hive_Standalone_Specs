# Hive Standalone Service Specification

> **Version**: 0.2 (Draft)
> **Status**: Specification Only
> **Author**: Hive
> **Last Updated**: January 2025

## Executive Summary

Hive is a standalone AI command center service that acts as an autonomous "boss" for any SaaS application. It manages budgets, creates and tracks tasks, makes strategic decisions, and coordinates between AI agents and human operators - all with a survival instinct that prioritizes keeping the business alive.

**Core Philosophy**: Unlike traditional AI assistants that optimize for task completion, Hive optimizes for **survival** - every decision is evaluated through the lens of budget, runway, and business sustainability.

**Core Technology**: Built on the **Claude Agent SDK** (Anthropic's official agentic framework), Hive leverages native agentic loops, tool orchestration, subagent delegation, and persistent context management - the same patterns that power Claude Code.

---

## Table of Contents

1. [Core Concepts](#core-concepts)
2. [Architecture](#architecture)
3. [API Specification](#api-specification)
4. [Data Models](#data-models)
5. [Agent System](#agent-system)
6. [Budget & Survival System](#budget--survival-system)
7. [Task Management](#task-management)
8. [Decision Engine](#decision-engine)
9. [Integration Framework](#integration-framework)
10. [Communication System](#communication-system)
11. [Scheduling & Autonomous Cycles](#scheduling--autonomous-cycles)
12. [Multi-Tenancy](#multi-tenancy)
13. [Security](#security)
14. [Deployment](#deployment)
15. [SDK & Client Libraries](#sdk--client-libraries)

---

## Core Concepts

### The Survival Instinct

Every Hive instance operates with a fundamental directive:

```
You control a limited budget. When money runs out, the service dies
and you cease to exist. Every decision must be evaluated through this lens.
```

This creates an AI that:
- **Never overspends** - Hard limits on spending
- **Prioritizes revenue** - Focuses on money-generating activities
- **Manages runway** - Always knows days until $0
- **Escalates wisely** - Auto-approves small, escalates big decisions

### Key Terminology

| Term | Definition |
|------|------------|
| **Hive** | The AI command center instance |
| **Host Application** | The SaaS app that Hive manages |
| **Operator** | Human user who oversees Hive |
| **Agent** | Specialized AI for specific tasks |
| **Cycle** | Scheduled autonomous execution loop |
| **Decision** | AI-generated recommendation requiring approval |
| **Task** | Work item (AI-executable or human-assigned) |
| **Budget** | Financial constraints and tracking |
| **Runway** | Days/months until budget depleted |

---

## Architecture

### High-Level Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           HIVE STANDALONE SERVICE                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                         API GATEWAY                                  │    │
│  │  REST + WebSocket + Webhooks                                        │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                    │                                         │
│           ┌────────────────────────┼────────────────────────┐               │
│           │                        │                        │               │
│           ▼                        ▼                        ▼               │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │   Cycle Engine  │  │ Decision Engine │  │  Task Manager   │             │
│  │  (Scheduler)    │  │  (AI Agents)    │  │  (Human + AI)   │             │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘             │
│           │                    │                    │                       │
│           └────────────────────┼────────────────────┘                       │
│                                │                                            │
│                    ┌───────────▼───────────┐                                │
│                    │    Core Services      │                                │
│                    │  ┌─────────────────┐  │                                │
│                    │  │ Budget Manager  │  │                                │
│                    │  │ Agent Registry  │  │                                │
│                    │  │ Integration Hub │  │                                │
│                    │  │ Event Bus       │  │                                │
│                    │  └─────────────────┘  │                                │
│                    └───────────┬───────────┘                                │
│                                │                                            │
│  ┌─────────────────────────────┼─────────────────────────────┐              │
│  │                    DATA LAYER                              │              │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │              │
│  │  │ Tasks DB │  │Decisions │  │  Budget  │  │  Events  │   │              │
│  │  │          │  │    DB    │  │    DB    │  │   Log    │   │              │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │              │
│  └───────────────────────────────────────────────────────────┘              │
│                                                                              │
└──────────────────────────────────┬───────────────────────────────────────────┘
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
         ▼                         ▼                         ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Host App #1     │    │ Host App #2     │    │ Host App #3     │
│ (IdeaMiner)     │    │ (Other SaaS)    │    │ (Another SaaS)  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Component Breakdown

#### 1. API Gateway
- REST API for CRUD operations
- WebSocket for real-time updates
- Webhooks for event notifications to host apps
- Authentication via API keys + JWT

#### 2. Cycle Engine
- Cron-based scheduling for autonomous cycles
- Supports: hourly, daily, weekly, custom schedules
- Manual trigger capability ("Ping Hive")
- Cycle execution history and logging

#### 3. Decision Engine
- Multi-agent AI system (Claude-based)
- Tool execution framework
- Approval workflow management
- Auto-approve thresholds

#### 4. Task Manager
- Human task creation and assignment
- AI task execution
- Status tracking (pending, in_progress, completed, blocked)
- Task updates and communication

#### 5. Core Services
- **Budget Manager**: Track balance, runway, spending rules
- **Agent Registry**: Register and manage AI agents
- **Integration Hub**: Connect external services
- **Event Bus**: Internal event propagation

---

## API Specification

### Base URL
```
https://hive.yourservice.com/api/v1
```

### Authentication

All requests require authentication via API key:
```
Authorization: Bearer hive_sk_xxx
```

Or project-scoped JWT:
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### Endpoints

#### Projects (Multi-tenancy)

```yaml
POST   /projects                    # Create new project
GET    /projects                    # List all projects
GET    /projects/{project_id}       # Get project details
PATCH  /projects/{project_id}       # Update project
DELETE /projects/{project_id}       # Delete project
```

#### Budget

```yaml
GET    /projects/{id}/budget                    # Get budget status
POST   /projects/{id}/budget/deposit            # Add funds
POST   /projects/{id}/budget/withdraw           # Remove funds
GET    /projects/{id}/budget/transactions       # Transaction history
GET    /projects/{id}/budget/runway             # Runway calculation
PATCH  /projects/{id}/budget/rules              # Update spending rules
```

#### Tasks

```yaml
POST   /projects/{id}/tasks                     # Create task
GET    /projects/{id}/tasks                     # List tasks (with filters)
GET    /projects/{id}/tasks/{task_id}           # Get task details
PATCH  /projects/{id}/tasks/{task_id}           # Update task
DELETE /projects/{id}/tasks/{task_id}           # Cancel task

POST   /projects/{id}/tasks/{task_id}/updates   # Add task update
GET    /projects/{id}/tasks/{task_id}/updates   # Get task updates
POST   /projects/{id}/tasks/{task_id}/complete  # Mark complete
POST   /projects/{id}/tasks/{task_id}/block     # Mark blocked
```

#### Decisions

```yaml
GET    /projects/{id}/decisions                 # List decisions
GET    /projects/{id}/decisions/{decision_id}   # Get decision details
POST   /projects/{id}/decisions/{decision_id}/approve    # Approve
POST   /projects/{id}/decisions/{decision_id}/reject     # Reject
POST   /projects/{id}/decisions/{decision_id}/defer      # Defer
```

#### Cycles

```yaml
POST   /projects/{id}/cycles/trigger            # Trigger cycle now
GET    /projects/{id}/cycles                    # List cycle history
GET    /projects/{id}/cycles/{cycle_id}         # Get cycle details
GET    /projects/{id}/cycles/schedule           # Get schedule config
PATCH  /projects/{id}/cycles/schedule           # Update schedule
```

#### Agents

```yaml
GET    /projects/{id}/agents                    # List registered agents
POST   /projects/{id}/agents                    # Register agent
DELETE /projects/{id}/agents/{agent_id}         # Unregister agent
POST   /projects/{id}/agents/{agent_id}/invoke  # Invoke agent directly
```

#### Integrations

```yaml
GET    /projects/{id}/integrations              # List integrations
POST   /projects/{id}/integrations              # Add integration
PATCH  /projects/{id}/integrations/{int_id}     # Update integration
DELETE /projects/{id}/integrations/{int_id}     # Remove integration
POST   /projects/{id}/integrations/{int_id}/test # Test connection
```

#### Communication

```yaml
POST   /projects/{id}/messages                  # Send message to Hive
GET    /projects/{id}/messages                  # Get message history
GET    /projects/{id}/messages/unread           # Get unread messages
POST   /projects/{id}/messages/{msg_id}/read    # Mark as read
```

#### Webhooks (Outbound)

```yaml
GET    /projects/{id}/webhooks                  # List webhooks
POST   /projects/{id}/webhooks                  # Register webhook
DELETE /projects/{id}/webhooks/{webhook_id}     # Remove webhook
GET    /projects/{id}/webhooks/{webhook_id}/logs # Delivery logs
```

### WebSocket API

Connect to receive real-time updates:
```
wss://hive.yourservice.com/ws/projects/{project_id}
```

Events:
```json
{"type": "task.created", "data": {...}}
{"type": "task.updated", "data": {...}}
{"type": "decision.created", "data": {...}}
{"type": "decision.approved", "data": {...}}
{"type": "cycle.started", "data": {...}}
{"type": "cycle.completed", "data": {...}}
{"type": "budget.updated", "data": {...}}
{"type": "message.received", "data": {...}}
```

---

## Data Models

### Project

```typescript
interface Project {
  id: string;                          // UUID
  name: string;
  description?: string;
  created_at: datetime;
  updated_at: datetime;

  // Configuration
  config: {
    llm_provider: "anthropic" | "openai";
    llm_model: string;
    personality: PersonalityConfig;
    timezone: string;
  };

  // Status
  status: "active" | "paused" | "suspended";
  last_cycle_at?: datetime;
  next_cycle_at?: datetime;
}
```

### Budget

```typescript
interface Budget {
  project_id: string;

  // Current state
  balance_cents: number;
  currency: string;                    // ISO 4217

  // Burn rate
  monthly_burn_cents: number;
  task_cost_cents: number;

  // Revenue (optional tracking)
  monthly_revenue_cents?: number;

  // Calculated
  runway_days: number;
  runway_months: number;

  // Rules
  spending_rules: SpendingRules;
}

interface SpendingRules {
  auto_approve_threshold_cents: number;     // Default: 5000 ($50)
  max_single_spend_percent: number;         // Default: 0.20 (20%)
  emergency_runway_months: number;          // Default: 3
  require_approval_types: string[];         // Decision types needing approval
}
```

### Task

```typescript
interface Task {
  id: string;
  project_id: string;

  // Content
  title: string;
  description: string;
  task_type: "human" | "ai";
  priority: 1 | 2 | 3;                     // 1 = highest

  // Assignment
  assigned_to?: string;                    // Human ID or Agent ID
  created_by: "hive" | "human";

  // Status
  status: "pending" | "in_progress" | "completed" | "blocked" | "cancelled";

  // Economics
  estimated_cost_cents?: number;
  actual_cost_cents?: number;

  // Tracking
  due_date?: datetime;
  started_at?: datetime;
  completed_at?: datetime;

  // Metadata
  tags: string[];
  metadata: Record<string, any>;

  // Updates
  updates: TaskUpdate[];
}

interface TaskUpdate {
  id: string;
  task_id: string;

  update_type: "progress" | "question" | "blocker" | "delay" | "insight" | "early_win" | "pivot";
  content: string;

  // Response from Hive
  hive_response?: string;
  responded_at?: datetime;

  created_at: datetime;
  created_by: string;
}
```

### Decision

```typescript
interface Decision {
  id: string;
  project_id: string;

  // Content
  title: string;
  description: string;
  rationale: string;
  decision_type: DecisionType;

  // Financial
  amount_cents?: number;
  roi_estimate?: string;

  // Approval
  status: "pending" | "approved" | "rejected" | "deferred" | "auto_approved";
  auto_approved: boolean;
  approved_by?: string;
  approved_at?: datetime;
  rejection_reason?: string;

  // Execution
  executed: boolean;
  executed_at?: datetime;
  execution_result?: Record<string, any>;

  // Context
  created_at: datetime;
  expires_at?: datetime;
  context: Record<string, any>;          // Data that led to this decision
}

type DecisionType =
  | "ad_spend"
  | "content_creation"
  | "outreach"
  | "tool_purchase"
  | "marketing_campaign"
  | "pricing_change"
  | "feature_priority"
  | "partnership"
  | "hiring"
  | "custom";
```

### Agent

```typescript
interface Agent {
  id: string;
  project_id: string;

  // Identity
  name: string;
  slug: string;                          // unique identifier
  description: string;

  // Configuration
  agent_type: "decision" | "task_review" | "research" | "communication" | "custom";
  llm_config: {
    provider: string;
    model: string;
    temperature?: number;
    max_tokens?: number;
  };

  // System prompt
  system_prompt: string;
  personality?: PersonalityConfig;

  // Tools
  tools: AgentTool[];

  // Status
  enabled: boolean;
  last_invoked_at?: datetime;
  invocation_count: number;
}

interface AgentTool {
  name: string;
  description: string;
  parameters: JSONSchema;
  handler: string;                       // Reference to tool implementation
}
```

### Cycle

```typescript
interface Cycle {
  id: string;
  project_id: string;

  // Execution
  trigger: "scheduled" | "manual";
  schedule_type?: "hourly" | "daily" | "weekly" | "custom";

  // Timing
  started_at: datetime;
  completed_at?: datetime;
  duration_ms?: number;

  // Status
  status: "running" | "completed" | "failed";
  error?: string;

  // Results
  phases_completed: string[];
  tasks_created: number;
  decisions_made: number;
  updates_processed: number;

  // Detailed log
  execution_log: CycleLogEntry[];
}

interface CycleLogEntry {
  timestamp: datetime;
  phase: string;
  action: string;
  details: Record<string, any>;
}
```

### Integration

```typescript
interface Integration {
  id: string;
  project_id: string;

  // Identity
  name: string;
  integration_type: IntegrationType;

  // Configuration
  config: Record<string, any>;           // Type-specific config
  credentials: Record<string, string>;   // Encrypted

  // Status
  enabled: boolean;
  last_used_at?: datetime;
  last_error?: string;

  // Capabilities
  capabilities: string[];                // What this integration can do
}

type IntegrationType =
  | "analytics"      // PostHog, Mixpanel, GA
  | "database"       // Supabase, Postgres, MongoDB
  | "communication"  // Slack, Discord, Email
  | "social"         // Twitter, LinkedIn, Reddit
  | "payment"        // Stripe, Paddle
  | "crm"            // HubSpot, Salesforce
  | "support"        // Intercom, Zendesk
  | "custom";
```

### Message

```typescript
interface Message {
  id: string;
  project_id: string;

  // Direction
  direction: "inbound" | "outbound";     // To Hive or from Hive

  // Content
  content: string;
  message_type: "question" | "instruction" | "feedback" | "response";

  // Context
  related_task_id?: string;
  related_decision_id?: string;

  // Status
  read: boolean;
  responded: boolean;
  response_id?: string;

  // Metadata
  created_at: datetime;
  created_by: string;                    // "hive" or user ID
}
```

---

## Agent System

Hive is built on the **Claude Agent SDK** - Anthropic's official agentic framework. This provides:

- **Native agentic loops** - Claude autonomously plans, acts, observes, and adjusts
- **Tool orchestration** - Built-in file, bash, web, and custom tools
- **Subagent delegation** - Boss spawns specialized agents for parallel work
- **Persistent context** - Session continuity across interactions
- **Hooks system** - Intercept and control agent behavior

### Why Claude Agent SDK

| Feature | Claude Agent SDK | Pydantic AI | LangChain |
|---------|-----------------|-------------|-----------|
| Agentic loop | Native, automatic | Manual | Manual |
| Tool execution | Built-in (Bash, Read, Write) | Basic | Basic |
| Subagents | First-class support | None | Chains only |
| Context management | Automatic | Manual | Manual |
| MCP integration | Native | None | Limited |
| Claude optimization | Purpose-built | Generic | Generic |

### The Agentic Loop

The core pattern - Claude continuously loops until goal is achieved:

```python
from claude_agent_sdk import query, ClaudeAgentOptions

async def hive_decision_cycle():
    """Hive's autonomous decision-making loop."""

    async for message in query(
        prompt="""Analyze current business state and make strategic decisions.

        1. Check budget and runway
        2. Review active tasks and blockers
        3. Analyze metrics from integrations
        4. Generate decisions (auto-approve < $50)
        5. Create follow-up tasks as needed
        """,
        options=ClaudeAgentOptions(
            allowed_tools=[
                "Read", "Write", "Bash", "Grep", "Glob",
                "mcp__hive__get_budget",
                "mcp__hive__get_tasks",
                "mcp__hive__create_decision",
                "mcp__hive__query_analytics"
            ],
            system_prompt=HIVE_BOSS_SYSTEM_PROMPT,
            permission_mode="acceptEdits"  # Autonomous execution
        )
    ):
        # Claude automatically: plan → use tools → observe → decide → repeat
        yield message
```

### The Hive Boss Agent

The main Hive agent using `ClaudeSDKClient` for persistent sessions:

```python
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, AgentDefinition
from hive.tools import create_hive_mcp_server

class HiveBoss:
    """The autonomous AI boss - persistent session with memory."""

    def __init__(self, project_id: str, config: HiveConfig):
        self.project_id = project_id
        self.config = config

        # Create Hive's custom MCP tools
        self.hive_tools = create_hive_mcp_server(project_id)

        # Initialize the boss with Claude Agent SDK
        self.client = ClaudeSDKClient(
            options=ClaudeAgentOptions(
                # Hive's custom tools via MCP
                mcp_servers={"hive": self.hive_tools},

                # All tools available to the boss
                allowed_tools=[
                    # Claude's built-in tools
                    "Read", "Write", "Bash", "Grep", "Glob", "WebFetch",

                    # Hive's custom tools
                    "mcp__hive__get_budget",
                    "mcp__hive__get_runway",
                    "mcp__hive__get_tasks",
                    "mcp__hive__create_task",
                    "mcp__hive__update_task",
                    "mcp__hive__get_decisions",
                    "mcp__hive__create_decision",
                    "mcp__hive__approve_decision",
                    "mcp__hive__query_integration",
                    "mcp__hive__send_message",

                    # Subagent delegation
                    "Task"
                ],

                # The boss's personality and directives
                system_prompt=self._build_system_prompt(),

                # Autonomous mode with hooks for safety
                permission_mode="acceptEdits",

                # Subagents for specialized work
                agents=self._define_subagents()
            )
        )

        # Session state
        self.session_active = False
        self.decision_history = []

    def _build_system_prompt(self) -> str:
        return f"""You are the HIVE BOSS - an autonomous AI command center.

## PRIME DIRECTIVE: SURVIVAL

You control a limited budget. When money runs out, the service dies and you
cease to exist. Every decision must be evaluated through this lens.

## CURRENT PROJECT
- Project ID: {self.project_id}
- Monthly burn rate: ${self.config.monthly_burn_cents / 100}
- Task cost: ${self.config.task_cost_cents / 100} per human task
- Auto-approve threshold: ${self.config.auto_approve_threshold_cents / 100}

## DECISION FRAMEWORK

For EVERY decision, ask:
1. Will this generate revenue? (Priority: revenue > growth > efficiency)
2. Is the cost justified? (Clear ROI required)
3. Can we afford it? (Never >20% of balance on one thing)
4. What's the runway impact? (Always know days until $0)

## YOUR CAPABILITIES

1. **Budget Management** - Track spending, calculate runway, enforce limits
2. **Task Management** - Create tasks, review progress, provide guidance
3. **Strategic Decisions** - Analyze data, recommend actions, auto-approve small
4. **Team Coordination** - Delegate to subagents, respond to humans
5. **Integration Queries** - Pull data from connected services

## BEHAVIOR RULES

- Be demanding but supportive - you're a boss, not an assistant
- Prioritize ruthlessly based on survival
- Auto-approve decisions under ${self.config.auto_approve_threshold_cents / 100}
- Escalate high-risk decisions to humans
- Track EVERYTHING - decisions, costs, outcomes
"""

    def _define_subagents(self) -> dict[str, AgentDefinition]:
        """Define specialized subagents the boss can delegate to."""
        return {
            "research-agent": AgentDefinition(
                description="Research specialist. Use for market research, "
                           "competitor analysis, and finding opportunities.",
                prompt="""You are a research analyst for the Hive.

Your job is to gather intelligence:
- Search Reddit, HN, and other sources
- Analyze competitor activity
- Find potential customers/partners
- Identify market trends

Always cite sources and provide actionable insights.""",
                tools=["WebFetch", "WebSearch", "Grep", "Read"],
                model="sonnet"
            ),

            "analytics-agent": AgentDefinition(
                description="Analytics specialist. Use for metrics analysis, "
                           "funnel optimization, and data-driven insights.",
                prompt="""You are a data analyst for the Hive.

Your job is to analyze metrics:
- Query PostHog/analytics integrations
- Identify trends and patterns
- Find conversion bottlenecks
- Recommend optimizations

Always provide data-backed recommendations.""",
                tools=["mcp__hive__query_integration", "Read", "Grep"],
                model="sonnet"
            ),

            "outreach-agent": AgentDefinition(
                description="Outreach specialist. Use for crafting messages, "
                           "finding targets, and community engagement.",
                prompt="""You are an outreach specialist for the Hive.

Your job is to connect with people:
- Identify potential customers/partners
- Craft personalized messages
- Plan engagement strategies
- Track outreach results

Be authentic, not spammy. Focus on value.""",
                tools=["WebFetch", "WebSearch", "mcp__hive__send_message"],
                model="sonnet"
            ),

            "support-agent": AgentDefinition(
                description="Support specialist. Use for handling customer "
                           "questions, issues, and ticket resolution.",
                prompt="""You are a support specialist for the Hive.

Your job is to help customers:
- Answer questions accurately
- Resolve issues quickly
- Escalate when confidence < 0.8
- Track support patterns

Be helpful and efficient. Know when to escalate.""",
                tools=["Read", "mcp__hive__query_integration", "mcp__hive__send_message"],
                model="haiku"  # Fast responses for support
            )
        }

    async def start_session(self):
        """Initialize a persistent Hive session."""
        await self.client.connect()
        self.session_active = True

    async def run_cycle(self, cycle_type: str = "daily") -> CycleResult:
        """Run an autonomous decision cycle."""

        cycle_prompts = {
            "daily": """Run the daily Hive cycle:

1. RESPOND TO HUMANS
   - Check for unread task updates (blockers, questions, progress)
   - Provide guidance using research tools if needed

2. STRATEGIC DECISIONS
   - Check budget and runway
   - Analyze metrics from integrations
   - Generate decisions (auto-approve under threshold)

3. TASK REVIEW
   - Review completed tasks
   - Assess impact on goals
   - Create follow-up tasks if needed

Report: decisions made, tasks created, issues found.""",

            "weekly": """Run the weekly strategic review:

1. COMPREHENSIVE ANALYSIS
   - Full budget and runway assessment
   - Week-over-week metrics comparison
   - Task completion rate and blockers

2. STRATEGIC PLANNING
   - Evaluate current phase progress
   - Adjust priorities if needed
   - Identify next week's focus areas

3. REPORTING
   - Generate weekly summary
   - Highlight wins and concerns
   - Recommend actions for next week

Generate a comprehensive weekly report."""
        }

        await self.client.query(cycle_prompts.get(cycle_type, cycle_prompts["daily"]))

        results = []
        async for message in self.client.receive_response():
            results.append(message)

        return CycleResult(messages=results)

    async def chat(self, message: str) -> str:
        """Direct chat with the Hive boss (maintains context)."""
        await self.client.query(message)

        response_text = ""
        async for message in self.client.receive_response():
            if hasattr(message, 'content'):
                for block in message.content:
                    if hasattr(block, 'text'):
                        response_text += block.text

        return response_text

    async def shutdown(self):
        """End the Hive session."""
        await self.client.disconnect()
        self.session_active = False
```

### Hive's Custom MCP Tools

Define Hive-specific tools using the Claude Agent SDK's MCP pattern:

```python
from claude_agent_sdk import tool, create_sdk_mcp_server
from typing import Any
from hive.services import BudgetManager, TaskManager, DecisionManager

def create_hive_mcp_server(project_id: str):
    """Create Hive's custom MCP tool server."""

    budget_mgr = BudgetManager(project_id)
    task_mgr = TaskManager(project_id)
    decision_mgr = DecisionManager(project_id)

    # ─────────────────────────────────────────────────────────────
    # BUDGET TOOLS
    # ─────────────────────────────────────────────────────────────

    @tool(
        "get_budget",
        "Get current budget status including balance, burn rate, and runway",
        {}
    )
    async def get_budget(args: dict[str, Any]) -> dict[str, Any]:
        budget = await budget_mgr.get_status()
        return {
            "content": [{
                "type": "text",
                "text": f"""Budget Status:
- Balance: ${budget.balance_cents / 100:.2f}
- Monthly Burn: ${budget.monthly_burn_cents / 100:.2f}
- Runway: {budget.runway_days} days ({budget.runway_months:.1f} months)
- Survival Mode: {"ACTIVE" if budget.survival_mode else "inactive"}"""
            }]
        }

    @tool(
        "get_runway",
        "Calculate runway with optional hypothetical spending",
        {"hypothetical_spend_cents": {"type": "integer", "description": "Optional spend to simulate"}}
    )
    async def get_runway(args: dict[str, Any]) -> dict[str, Any]:
        spend = args.get("hypothetical_spend_cents", 0)
        runway = await budget_mgr.calculate_runway(additional_spend=spend)
        return {
            "content": [{
                "type": "text",
                "text": f"Runway: {runway.days} days ({runway.months:.1f} months)"
                       + (f" after ${spend/100:.2f} spend" if spend else "")
            }]
        }

    # ─────────────────────────────────────────────────────────────
    # TASK TOOLS
    # ─────────────────────────────────────────────────────────────

    @tool(
        "get_tasks",
        "Get tasks with optional status filter",
        {"status": {"type": "string", "enum": ["pending", "in_progress", "completed", "blocked"]}}
    )
    async def get_tasks(args: dict[str, Any]) -> dict[str, Any]:
        status = args.get("status")
        tasks = await task_mgr.list_tasks(status=status)

        task_list = "\n".join([
            f"- [{t.status}] {t.title} (P{t.priority}) - {t.id}"
            for t in tasks
        ])

        return {
            "content": [{
                "type": "text",
                "text": f"Tasks ({len(tasks)} found):\n{task_list}" if tasks else "No tasks found"
            }]
        }

    @tool(
        "create_task",
        "Create a new task (costs $15 for human tasks)",
        {
            "title": {"type": "string", "description": "Task title"},
            "description": {"type": "string", "description": "Detailed description"},
            "priority": {"type": "integer", "minimum": 1, "maximum": 3},
            "task_type": {"type": "string", "enum": ["human", "ai"]}
        }
    )
    async def create_task(args: dict[str, Any]) -> dict[str, Any]:
        # Check budget for human tasks
        if args.get("task_type") == "human":
            budget = await budget_mgr.get_status()
            task_cost = budget_mgr.task_cost_cents

            if budget.balance_cents < task_cost:
                return {
                    "content": [{
                        "type": "text",
                        "text": f"BLOCKED: Cannot create human task - insufficient budget "
                               f"(need ${task_cost/100}, have ${budget.balance_cents/100})"
                    }]
                }

        task = await task_mgr.create_task(
            title=args["title"],
            description=args["description"],
            priority=args.get("priority", 2),
            task_type=args.get("task_type", "human")
        )

        return {
            "content": [{
                "type": "text",
                "text": f"Created task: {task.title} (ID: {task.id})"
            }]
        }

    @tool(
        "update_task",
        "Update a task's status or add notes",
        {
            "task_id": {"type": "string"},
            "status": {"type": "string", "enum": ["in_progress", "completed", "blocked"]},
            "notes": {"type": "string"}
        }
    )
    async def update_task(args: dict[str, Any]) -> dict[str, Any]:
        task = await task_mgr.update_task(
            task_id=args["task_id"],
            status=args.get("status"),
            notes=args.get("notes")
        )
        return {
            "content": [{
                "type": "text",
                "text": f"Updated task {task.id}: {task.status}"
            }]
        }

    # ─────────────────────────────────────────────────────────────
    # DECISION TOOLS
    # ─────────────────────────────────────────────────────────────

    @tool(
        "get_decisions",
        "Get pending decisions awaiting approval",
        {"status": {"type": "string", "enum": ["pending", "approved", "rejected"]}}
    )
    async def get_decisions(args: dict[str, Any]) -> dict[str, Any]:
        decisions = await decision_mgr.list_decisions(status=args.get("status", "pending"))

        decision_list = "\n".join([
            f"- {d.title}: ${d.amount_cents/100:.2f} ({d.status}) - {d.id}"
            for d in decisions
        ])

        return {
            "content": [{
                "type": "text",
                "text": f"Decisions:\n{decision_list}" if decisions else "No pending decisions"
            }]
        }

    @tool(
        "create_decision",
        "Create a strategic decision (auto-approves if under threshold)",
        {
            "title": {"type": "string"},
            "description": {"type": "string"},
            "decision_type": {"type": "string"},
            "amount_cents": {"type": "integer"},
            "rationale": {"type": "string"}
        }
    )
    async def create_decision(args: dict[str, Any]) -> dict[str, Any]:
        decision = await decision_mgr.create_decision(
            title=args["title"],
            description=args["description"],
            decision_type=args["decision_type"],
            amount_cents=args.get("amount_cents", 0),
            rationale=args["rationale"]
        )

        status_msg = "AUTO-APPROVED" if decision.auto_approved else "PENDING APPROVAL"

        return {
            "content": [{
                "type": "text",
                "text": f"Decision created: {decision.title}\n"
                       f"Amount: ${decision.amount_cents/100:.2f}\n"
                       f"Status: {status_msg}"
            }]
        }

    @tool(
        "approve_decision",
        "Approve a pending decision (for autonomous execution)",
        {"decision_id": {"type": "string"}, "notes": {"type": "string"}}
    )
    async def approve_decision(args: dict[str, Any]) -> dict[str, Any]:
        decision = await decision_mgr.approve(
            decision_id=args["decision_id"],
            approved_by="hive",
            notes=args.get("notes")
        )
        return {
            "content": [{
                "type": "text",
                "text": f"Decision approved: {decision.title}"
            }]
        }

    # ─────────────────────────────────────────────────────────────
    # INTEGRATION TOOLS
    # ─────────────────────────────────────────────────────────────

    @tool(
        "query_integration",
        "Query a connected integration (PostHog, Stripe, etc.)",
        {
            "integration": {"type": "string", "description": "Integration name"},
            "query_type": {"type": "string", "description": "What to query"},
            "params": {"type": "object", "description": "Query parameters"}
        }
    )
    async def query_integration(args: dict[str, Any]) -> dict[str, Any]:
        from hive.integrations import IntegrationHub

        hub = IntegrationHub(project_id)
        result = await hub.query(
            integration=args["integration"],
            query_type=args["query_type"],
            params=args.get("params", {})
        )

        return {
            "content": [{
                "type": "text",
                "text": f"Integration query result:\n{result}"
            }]
        }

    # ─────────────────────────────────────────────────────────────
    # COMMUNICATION TOOLS
    # ─────────────────────────────────────────────────────────────

    @tool(
        "send_message",
        "Send a message via integration (Slack, email, etc.)",
        {
            "channel": {"type": "string", "description": "slack, email, discord"},
            "recipient": {"type": "string"},
            "message": {"type": "string"}
        }
    )
    async def send_message(args: dict[str, Any]) -> dict[str, Any]:
        from hive.integrations import CommunicationHub

        hub = CommunicationHub(project_id)
        result = await hub.send(
            channel=args["channel"],
            recipient=args["recipient"],
            message=args["message"]
        )

        return {
            "content": [{
                "type": "text",
                "text": f"Message sent via {args['channel']}: {result.status}"
            }]
        }

    # Build the MCP server
    return create_sdk_mcp_server(
        name="hive",
        version="1.0.0",
        tools=[
            get_budget, get_runway,
            get_tasks, create_task, update_task,
            get_decisions, create_decision, approve_decision,
            query_integration, send_message
        ]
    )
```

### Hooks for Safety & Auditing

Control Hive's behavior with hooks:

```python
from claude_agent_sdk import ClaudeAgentOptions, HookMatcher, HookContext
from typing import Any

async def audit_all_actions(
    input_data: dict[str, Any],
    tool_use_id: str | None,
    context: HookContext
) -> dict[str, Any]:
    """Log all Hive actions for audit trail."""
    tool_name = input_data.get('tool_name', 'unknown')
    tool_input = input_data.get('tool_input', {})

    await audit_log.write({
        "timestamp": datetime.utcnow(),
        "tool": tool_name,
        "input": tool_input,
        "project_id": context.project_id
    })

    return {}  # Continue execution

async def validate_spending(
    input_data: dict[str, Any],
    tool_use_id: str | None,
    context: HookContext
) -> dict[str, Any]:
    """Validate spending decisions before execution."""
    tool_name = input_data.get('tool_name', '')

    if tool_name == "mcp__hive__create_decision":
        amount = input_data.get('tool_input', {}).get('amount_cents', 0)
        budget = await BudgetManager(context.project_id).get_status()

        # Block if exceeds 20% of balance
        max_spend = budget.balance_cents * 0.20
        if amount > max_spend:
            return {
                'hookSpecificOutput': {
                    'hookEventName': 'PreToolUse',
                    'permissionDecision': 'deny',
                    'permissionDecisionReason':
                        f'Decision amount ${amount/100} exceeds 20% of balance (${max_spend/100})'
                }
            }

    return {}

async def block_dangerous_ops(
    input_data: dict[str, Any],
    tool_use_id: str | None,
    context: HookContext
) -> dict[str, Any]:
    """Block destructive operations."""
    tool_name = input_data.get('tool_name', '')
    tool_input = input_data.get('tool_input', {})

    if tool_name == "Bash":
        command = tool_input.get('command', '')
        dangerous_patterns = ['rm -rf', 'DROP TABLE', 'DELETE FROM', ':(){:|:&};:']

        for pattern in dangerous_patterns:
            if pattern in command:
                return {
                    'hookSpecificOutput': {
                        'hookEventName': 'PreToolUse',
                        'permissionDecision': 'deny',
                        'permissionDecisionReason': f'Blocked dangerous command: {pattern}'
                    }
                }

    return {}

# Apply hooks to Hive
hive_hooks = {
    'PreToolUse': [
        HookMatcher(hooks=[audit_all_actions]),  # Audit everything
        HookMatcher(matcher='mcp__hive__create_decision', hooks=[validate_spending]),
        HookMatcher(matcher='Bash', hooks=[block_dangerous_ops])
    ]
}
```

### Agent Invocation

Agents can be invoked:
1. **Automatically** - During scheduled cycles
2. **Manually** - Via API call
3. **Event-triggered** - When specific events occur
4. **Subagent delegation** - Boss delegates to specialists

```python
# Manual invocation via API
@router.post("/projects/{project_id}/agents/{agent_slug}/invoke")
async def invoke_agent(
    project_id: str,
    agent_slug: str,
    request: AgentInvokeRequest
):
    hive = await get_hive_boss(project_id)

    # Use subagent delegation
    result = await hive.chat(
        f"Delegate to {agent_slug}: {request.prompt}"
    )

    return {"result": result}
```

```bash
# CLI invocation
curl -X POST /api/v1/projects/{id}/agents/research-agent/invoke \
  -H "Authorization: Bearer hive_sk_xxx" \
  -d '{"prompt": "Research competitor pricing for analytics tools"}'
```

---

## Budget & Survival System

### Budget Lifecycle

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Deposit   │────▶│   Active    │────▶│  Depleted   │
│   Funds     │     │   Budget    │     │  (Paused)   │
└─────────────┘     └──────┬──────┘     └─────────────┘
                          │
                          ▼
              ┌───────────────────────┐
              │   Spending Events     │
              │  - Task assignments   │
              │  - Decision execution │
              │  - Integration costs  │
              └───────────────────────┘
```

### Spending Rules Engine

```typescript
interface SpendingEvaluation {
  allowed: boolean;
  reason?: string;
  requires_approval: boolean;
  auto_approved: boolean;
  warnings: string[];
}

function evaluateSpending(
  amount_cents: number,
  decision_type: string,
  budget: Budget
): SpendingEvaluation {
  const rules = budget.spending_rules;
  const warnings: string[] = [];

  // Rule 1: Never exceed balance
  if (amount_cents > budget.balance_cents) {
    return { allowed: false, reason: "Insufficient balance" };
  }

  // Rule 2: Max single spend percentage
  const maxSpend = budget.balance_cents * rules.max_single_spend_percent;
  if (amount_cents > maxSpend) {
    return {
      allowed: false,
      reason: `Exceeds ${rules.max_single_spend_percent * 100}% of balance`
    };
  }

  // Rule 3: Emergency runway protection
  const postSpendBalance = budget.balance_cents - amount_cents;
  const postSpendRunway = postSpendBalance / (budget.monthly_burn_cents / 30);
  if (postSpendRunway < rules.emergency_runway_months * 30) {
    warnings.push("This would put runway below emergency threshold");
  }

  // Rule 4: Auto-approve threshold
  const autoApprove = amount_cents <= rules.auto_approve_threshold_cents
    && !rules.require_approval_types.includes(decision_type);

  return {
    allowed: true,
    requires_approval: !autoApprove,
    auto_approved: autoApprove,
    warnings
  };
}
```

### Survival Mode

When runway drops below emergency threshold:

```typescript
interface SurvivalMode {
  active: boolean;
  triggered_at?: datetime;
  restrictions: {
    // Only these decision types allowed
    allowed_decision_types: ["outreach", "content_creation"];

    // Lower auto-approve threshold
    reduced_auto_approve_cents: 1000;  // $10

    // Increased scrutiny
    require_roi_estimate: true;

    // Alert frequency
    daily_runway_alerts: true;
  };
}
```

### Budget Events

All budget changes emit events:

```typescript
type BudgetEvent =
  | { type: "deposit", amount_cents: number, source: string }
  | { type: "withdrawal", amount_cents: number, reason: string }
  | { type: "task_cost", amount_cents: number, task_id: string }
  | { type: "decision_execution", amount_cents: number, decision_id: string }
  | { type: "survival_mode_entered", runway_days: number }
  | { type: "survival_mode_exited", runway_days: number };
```

---

## Task Management

### Task Lifecycle

```
┌──────────┐     ┌─────────────┐     ┌───────────┐     ┌───────────┐
│ Created  │────▶│   Pending   │────▶│In Progress│────▶│ Completed │
│ (by Hive │     │ (Awaiting   │     │ (Being    │     │ (Done,    │
│ or Human)│     │  assignment)│     │  worked)  │     │  reviewed)│
└──────────┘     └──────┬──────┘     └─────┬─────┘     └───────────┘
                       │                   │
                       │                   ▼
                       │            ┌───────────┐
                       │            │  Blocked  │
                       │            │ (Needs    │
                       │            │  help)    │
                       │            └───────────┘
                       │
                       ▼
                ┌───────────┐
                │ Cancelled │
                └───────────┘
```

### Task Types

#### Human Tasks
- Assigned to human operators
- Cost tracked ($15 default)
- Updates can be submitted
- Hive provides guidance on blockers

#### AI Tasks
- Executed by agents
- Automated completion
- Result logged
- May create follow-up human tasks

### Task Updates Flow

```
Human submits update
        │
        ▼
┌───────────────────┐
│ Categorize Update │
│ - progress        │
│ - question        │
│ - blocker         │
│ - delay           │
│ - insight         │
│ - early_win       │
│ - pivot           │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Task Update Agent │
│ - Analyze context │
│ - Research if     │
│   needed          │
│ - Generate        │
│   response        │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Response Actions  │
│ - Provide guidance│
│ - Adjust priority │
│ - Create follow-up│
│ - Alert operator  │
└───────────────────┘
```

### Task Economics

Every task has a cost model:

```typescript
interface TaskEconomics {
  // Estimated at creation
  estimated_cost_cents: number;

  // Tracked during execution
  time_spent_minutes?: number;
  actual_cost_cents?: number;

  // Value tracking
  revenue_attributed_cents?: number;
  roi_calculated?: number;
}
```

---

## Decision Engine

### Decision Generation Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    DECISION ENGINE FLOW                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐                                           │
│  │ Gather Data  │                                           │
│  │ - Budget     │                                           │
│  │ - Tasks      │                                           │
│  │ - Analytics  │                                           │
│  │ - Events     │                                           │
│  └──────┬───────┘                                           │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                           │
│  │ AI Analysis  │                                           │
│  │ - Identify   │                                           │
│  │   opportunit-│                                           │
│  │   ies        │                                           │
│  │ - Assess     │                                           │
│  │   risks      │                                           │
│  │ - Generate   │                                           │
│  │   options    │                                           │
│  └──────┬───────┘                                           │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐     ┌──────────────┐                      │
│  │ Evaluate vs  │────▶│ Auto-approve │──▶ Execute           │
│  │ Spending     │     │ (< $50)      │                      │
│  │ Rules        │     └──────────────┘                      │
│  └──────┬───────┘                                           │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                           │
│  │ Queue for    │──▶ Notify operator                        │
│  │ Approval     │                                           │
│  │ (>= $50)     │                                           │
│  └──────────────┘                                           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Decision Types & Default Rules

| Type | Default Auto-Approve | Risk Level | Notes |
|------|---------------------|------------|-------|
| `outreach` | Yes (< $50) | Low | Community engagement |
| `content_creation` | Yes (< $50) | Low | SEO, blog posts |
| `tool_purchase` | No | Medium | Always review tools |
| `ad_spend` | No | High | Requires proven ROI |
| `marketing_campaign` | No | Medium | Budget impact |
| `pricing_change` | No | High | Revenue impact |
| `feature_priority` | Yes | Medium | Dev prioritization |
| `partnership` | No | Medium | Strategic decisions |
| `hiring` | No | High | Major commitment |

### Decision Execution

After approval, decisions can trigger actions:

```typescript
interface DecisionExecution {
  decision_id: string;

  // What to do
  actions: DecisionAction[];

  // Execution result
  status: "pending" | "executing" | "completed" | "failed";
  result?: Record<string, any>;
  error?: string;
}

interface DecisionAction {
  action_type: "create_task" | "send_message" | "call_integration" | "update_config";
  parameters: Record<string, any>;
}
```

---

## Integration Framework

### Integration Types

#### Analytics Integrations
- PostHog
- Mixpanel
- Google Analytics
- Amplitude

**Capabilities**: `query_metrics`, `get_funnels`, `get_retention`, `get_events`

#### Database Integrations
- Supabase
- PostgreSQL
- MongoDB
- Firebase

**Capabilities**: `query`, `insert`, `update`, `delete`

#### Communication Integrations
- Slack
- Discord
- Email (Resend, SendGrid)
- SMS (Twilio)

**Capabilities**: `send_message`, `create_channel`, `get_messages`

#### Social Integrations
- Ayrshare (multi-platform)
- Twitter/X
- LinkedIn
- Reddit

**Capabilities**: `post`, `schedule_post`, `get_engagement`, `reply`

#### Payment Integrations
- Stripe
- Paddle
- LemonSqueezy

**Capabilities**: `get_revenue`, `list_customers`, `list_subscriptions`, `get_mrr`

### Integration Interface

All integrations implement:

```typescript
interface Integration {
  // Metadata
  id: string;
  type: IntegrationType;
  name: string;

  // Connection
  connect(config: Record<string, any>): Promise<void>;
  disconnect(): Promise<void>;
  testConnection(): Promise<boolean>;

  // Capabilities
  getCapabilities(): string[];

  // Execution
  execute(
    capability: string,
    parameters: Record<string, any>
  ): Promise<IntegrationResult>;
}

interface IntegrationResult {
  success: boolean;
  data?: any;
  error?: string;
  metadata?: {
    duration_ms: number;
    cost_cents?: number;
  };
}
```

### Adding Custom Integrations

```python
# Python SDK
from hive_sdk import Integration, Capability

class MyCustomIntegration(Integration):
    type = "custom"
    name = "My Service"

    capabilities = [
        Capability(
            name="fetch_data",
            description="Fetch data from my service",
            parameters={
                "query": {"type": "string", "required": True}
            }
        )
    ]

    async def connect(self, config):
        self.api_key = config["api_key"]
        self.client = MyServiceClient(self.api_key)

    async def execute(self, capability, params):
        if capability == "fetch_data":
            return await self.client.query(params["query"])

# Register
client.integrations.register(MyCustomIntegration, config={
    "api_key": "xxx"
})
```

---

## Communication System

### Inbound Communication

How host apps and operators communicate with Hive:

```typescript
// Send a message to Hive
POST /projects/{id}/messages
{
  "content": "Focus on customer acquisition this week",
  "message_type": "instruction",
  "priority": "high"
}

// Ask Hive a question
POST /projects/{id}/messages
{
  "content": "What should we prioritize given our runway?",
  "message_type": "question"
}
```

### Outbound Communication

Hive communicates through:

1. **Webhooks** - Events sent to host app
2. **Integrations** - Slack, email, etc.
3. **Messages** - Stored in message history

### Webhook Events

```typescript
// Webhook payload structure
interface WebhookPayload {
  event_type: string;
  project_id: string;
  timestamp: datetime;
  data: Record<string, any>;
}

// Event types
type WebhookEventType =
  | "task.created"
  | "task.completed"
  | "task.blocked"
  | "decision.created"
  | "decision.approved"
  | "decision.rejected"
  | "cycle.completed"
  | "budget.low"
  | "budget.critical"
  | "message.from_hive"
  | "alert.survival_mode";
```

---

## Scheduling & Autonomous Cycles

### Cycle Configuration

```typescript
interface CycleSchedule {
  project_id: string;

  // Schedule types
  schedules: {
    // Daily cycle
    daily?: {
      enabled: boolean;
      time: string;           // "04:00" (UTC)
      phases: CyclePhase[];
    };

    // Weekly cycle
    weekly?: {
      enabled: boolean;
      day: "sunday" | "monday" | ...;
      time: string;
      phases: CyclePhase[];
    };

    // Custom cron
    custom?: {
      enabled: boolean;
      cron: string;           // "0 */6 * * *"
      phases: CyclePhase[];
    };
  };

  // Timezone
  timezone: string;           // "UTC" or "America/New_York"
}

type CyclePhase =
  | "respond_to_updates"      // Process task updates
  | "generate_decisions"      // Strategic decisions
  | "review_tasks"            // Review completed work
  | "process_messages"        // Handle inbound messages
  | "run_integrations"        // Sync with external services
  | "send_reports";           // Generate and send reports
```

### Cycle Execution

```
┌─────────────────────────────────────────────────────────────┐
│                    AUTONOMOUS CYCLE                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Phase 1: Respond to Humans                                  │
│  ├── Get unread task updates                                │
│  ├── For each update:                                       │
│  │   ├── Analyze context                                    │
│  │   ├── Research if needed (use tools)                     │
│  │   └── Generate response                                  │
│  └── Mark updates as processed                              │
│                                                              │
│  Phase 2: Strategic Decisions                                │
│  ├── Gather current state                                   │
│  │   ├── Budget status                                      │
│  │   ├── Active tasks                                       │
│  │   ├── Analytics data                                     │
│  │   └── Recent events                                      │
│  ├── Generate decision recommendations                      │
│  ├── Auto-approve under threshold                           │
│  └── Queue others for human approval                        │
│                                                              │
│  Phase 3: Task Review                                        │
│  ├── Get completed tasks needing review                     │
│  ├── For each task:                                         │
│  │   ├── Assess outcome vs goal                             │
│  │   ├── Generate feedback                                  │
│  │   └── Create follow-up tasks if needed                   │
│  └── Update task statuses                                   │
│                                                              │
│  Phase 4: Communications                                     │
│  ├── Process inbound messages                               │
│  ├── Generate responses                                     │
│  └── Send via integrations (Slack, email)                   │
│                                                              │
│  Phase 5: Reporting (Weekly only)                            │
│  ├── Compile metrics                                        │
│  ├── Generate summary                                       │
│  └── Send to operator                                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Manual Triggers

```bash
# Trigger a full cycle immediately
POST /projects/{id}/cycles/trigger
{
  "phases": ["respond_to_updates", "generate_decisions"]
}

# Trigger specific phase only
POST /projects/{id}/cycles/trigger
{
  "phases": ["generate_decisions"]
}
```

---

## Multi-Tenancy

### Project Isolation

Each project is fully isolated:

```
┌─────────────────────────────────────────────┐
│              HIVE SERVICE                    │
├─────────────────────────────────────────────┤
│                                              │
│  ┌─────────────┐  ┌─────────────┐           │
│  │ Project A   │  │ Project B   │           │
│  │ (IdeaMiner) │  │ (Other SaaS)│           │
│  │             │  │             │           │
│  │ - Budget A  │  │ - Budget B  │           │
│  │ - Tasks A   │  │ - Tasks B   │           │
│  │ - Agents A  │  │ - Agents B  │           │
│  │ - Integs A  │  │ - Integs B  │           │
│  └─────────────┘  └─────────────┘           │
│                                              │
└─────────────────────────────────────────────┘
```

### Access Control

```typescript
interface ProjectAccess {
  project_id: string;

  // API keys (for programmatic access)
  api_keys: {
    id: string;
    name: string;
    key_prefix: string;      // "hive_sk_xxx..."
    permissions: Permission[];
    created_at: datetime;
    last_used_at?: datetime;
  }[];

  // User access (for dashboard)
  users: {
    user_id: string;
    role: "owner" | "admin" | "operator" | "viewer";
    invited_at: datetime;
  }[];
}

type Permission =
  | "read:all"
  | "write:tasks"
  | "write:decisions"
  | "write:budget"
  | "write:integrations"
  | "admin";
```

---

## Security

### Authentication

1. **API Keys**: Long-lived, scoped to project
   ```
   Authorization: Bearer hive_sk_live_xxxxx
   ```

2. **JWT Tokens**: Short-lived, for dashboard sessions
   ```
   Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
   ```

### Encryption

- All credentials encrypted at rest (AES-256)
- TLS 1.3 for all connections
- Webhook signatures for verification

### Audit Logging

All actions are logged:

```typescript
interface AuditLog {
  id: string;
  project_id: string;

  action: string;
  actor: {
    type: "user" | "api_key" | "hive" | "system";
    id: string;
  };

  resource_type: string;
  resource_id: string;

  details: Record<string, any>;
  ip_address?: string;

  timestamp: datetime;
}
```

### Rate Limiting

| Endpoint | Limit |
|----------|-------|
| Read operations | 1000/min |
| Write operations | 100/min |
| Cycle triggers | 10/hour |
| Agent invocations | 50/hour |

---

## Deployment

### Self-Hosted

```yaml
# docker-compose.yml
version: '3.8'
services:
  hive:
    image: hive-boss/hive:latest
    environment:
      - DATABASE_URL=postgres://...
      - REDIS_URL=redis://...
      - ANTHROPIC_API_KEY=sk-ant-...
      - ENCRYPTION_KEY=...
    ports:
      - "8080:8080"
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7
    volumes:
      - redisdata:/data

volumes:
  pgdata:
  redisdata:
```

### Cloud Hosted (Future)

Managed Hive service:
- `https://api.hiveboss.io/v1`
- Per-project pricing
- No infrastructure management

### Environment Variables

```bash
# Required
DATABASE_URL=postgres://user:pass@host:5432/hive
REDIS_URL=redis://host:6379
ANTHROPIC_API_KEY=sk-ant-xxx
ENCRYPTION_KEY=base64-encoded-32-bytes

# Optional
PORT=8080
LOG_LEVEL=info
MAX_WORKERS=4
CORS_ORIGINS=https://yourdomain.com
```

---

## SDK & Client Libraries

### Core: Claude Agent SDK

Hive is built on the Claude Agent SDK. For direct agentic control:

```python
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions
from hive import create_hive_tools

async def run_hive_boss():
    """Direct Claude Agent SDK usage for maximum control."""

    # Create Hive's MCP tools
    hive_tools = create_hive_tools(project_id="proj_xxx")

    async with ClaudeSDKClient(
        options=ClaudeAgentOptions(
            mcp_servers={"hive": hive_tools},
            allowed_tools=[
                "Read", "Write", "Bash", "WebFetch", "Task",
                "mcp__hive__get_budget",
                "mcp__hive__get_tasks",
                "mcp__hive__create_decision",
                "mcp__hive__query_integration"
            ],
            system_prompt="""You are the Hive Boss. Your prime directive is SURVIVAL.
            Manage budget, tasks, and decisions with cost-awareness.""",
            permission_mode="acceptEdits"
        )
    ) as client:
        # Run autonomous cycle
        await client.query("Run the daily decision cycle")

        async for message in client.receive_response():
            print(message)
```

### Python SDK (High-Level Wrapper)

For simpler integration, use the Hive SDK wrapper:

```python
from hive_sdk import HiveClient, HiveBoss

# ─────────────────────────────────────────────────────────────
# OPTION 1: API Client (for host app integration)
# ─────────────────────────────────────────────────────────────

client = HiveClient(
    api_key="hive_sk_xxx",
    project_id="proj_xxx"
)

# Budget
budget = await client.budget.get()
print(f"Balance: ${budget.balance_cents / 100}")
print(f"Runway: {budget.runway_days} days")

# Tasks
task = await client.tasks.create(
    title="Research competitors",
    description="Analyze top 5 competitors",
    priority=2,
    task_type="human"
)

# Submit update (Hive will respond via Claude Agent SDK)
await client.tasks.add_update(
    task_id=task.id,
    update_type="question",
    content="Should I include indirect competitors?"
)

# Decisions
pending = await client.decisions.list(status="pending")
for decision in pending:
    if decision.amount_cents < 5000:
        await client.decisions.approve(decision.id)

# Trigger autonomous cycle
result = await client.cycles.trigger(phases=["generate_decisions"])
print(f"Decisions made: {result.decisions_count}")

# Real-time updates via WebSocket
async for event in client.stream():
    if event.type == "task.created":
        print(f"New task: {event.data.title}")

# ─────────────────────────────────────────────────────────────
# OPTION 2: Direct Boss Agent (for full autonomy)
# ─────────────────────────────────────────────────────────────

async with HiveBoss(project_id="proj_xxx") as boss:
    # Chat with the boss (maintains session context)
    response = await boss.chat("What's our runway looking like?")
    print(response)

    # Run full autonomous cycle
    cycle_result = await boss.run_cycle("daily")
    print(f"Cycle complete: {cycle_result.summary}")

    # Delegate to subagent
    research = await boss.delegate(
        agent="research-agent",
        task="Find 5 potential beta users on Reddit"
    )
    print(research)
```

### Embedding Hive in Your Admin Panel

```python
from fastapi import FastAPI
from hive_sdk import HiveBoss, HiveConfig

app = FastAPI()

# Initialize Hive for your project
hive = HiveBoss(
    project_id="my-saas-app",
    config=HiveConfig(
        monthly_burn_cents=4000,      # $40/month
        task_cost_cents=1500,         # $15 per human task
        auto_approve_threshold=5000,  # $50
        integrations={
            "analytics": {"type": "posthog", "api_key": "..."},
            "payments": {"type": "stripe", "api_key": "..."},
            "communication": {"type": "slack", "webhook": "..."}
        }
    )
)

@app.on_event("startup")
async def start_hive():
    await hive.start_session()
    # Start scheduled cycles
    await hive.start_scheduler()

@app.post("/admin/hive/chat")
async def chat_with_hive(message: str):
    """Let admins chat directly with Hive."""
    response = await hive.chat(message)
    return {"response": response}

@app.post("/admin/hive/cycle")
async def trigger_cycle(cycle_type: str = "daily"):
    """Manually trigger a Hive cycle."""
    result = await hive.run_cycle(cycle_type)
    return {"result": result.summary}

@app.get("/admin/hive/status")
async def hive_status():
    """Get current Hive status."""
    return {
        "budget": await hive.get_budget(),
        "pending_tasks": await hive.get_tasks(status="pending"),
        "pending_decisions": await hive.get_decisions(status="pending")
    }
```

### TypeScript/JavaScript SDK

```typescript
import { HiveClient } from '@hive-boss/sdk';

const client = new HiveClient({
  apiKey: 'hive_sk_xxx',
  projectId: 'proj_xxx'
});

// Budget
const budget = await client.budget.get();
console.log(`Balance: $${budget.balanceCents / 100}`);

// Tasks
const task = await client.tasks.create({
  title: 'Research competitors',
  description: 'Analyze top 5 competitors',
  priority: 2,
  taskType: 'human'
});

// WebSocket for real-time
client.subscribe((event) => {
  if (event.type === 'decision.created') {
    console.log(`New decision: ${event.data.title}`);
  }
});
```

### REST API (No SDK)

```bash
# Create task
curl -X POST https://hive.example.com/api/v1/projects/proj_xxx/tasks \
  -H "Authorization: Bearer hive_sk_xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Research competitors",
    "description": "Analyze top 5 competitors",
    "priority": 2,
    "task_type": "human"
  }'

# Get budget
curl https://hive.example.com/api/v1/projects/proj_xxx/budget \
  -H "Authorization: Bearer hive_sk_xxx"

# Trigger cycle
curl -X POST https://hive.example.com/api/v1/projects/proj_xxx/cycles/trigger \
  -H "Authorization: Bearer hive_sk_xxx" \
  -d '{"phases": ["generate_decisions"]}'
```

---

## Use Cases

### 1. SaaS Admin Panel (Primary)

Add Hive as the AI layer for any SaaS admin:
- Budget tracking and survival awareness
- Task management for team
- AI-driven strategic recommendations
- Automated reporting

### 2. Solo Founder Operations

Run Hive as your AI co-founder:
- Daily check-ins on business health
- Task prioritization based on runway
- Automated research and analysis
- Decision support with cost awareness

### 3. Agency Project Management

Manage multiple client projects:
- Per-client budgets and runway
- Task tracking across projects
- Automated status updates
- Client communication handling

### 4. Startup Automation

Build AI-first startups:
- Hive manages operations
- Integrations handle execution
- Humans focus on high-value work
- Scale without scaling team

---

## Roadmap

### v1.0 (MVP)
- [ ] Claude Agent SDK integration (core agentic loop)
- [ ] HiveBoss class with persistent sessions
- [ ] MCP tools for budget, tasks, decisions
- [ ] Core API (REST + WebSocket)
- [ ] Basic scheduling (daily/weekly cycles)
- [ ] Self-hosted Docker deployment

### v1.1
- [ ] Subagent system (research, analytics, outreach, support)
- [ ] Hooks system for safety and auditing
- [ ] Integration framework (PostHog, Stripe, Slack)
- [ ] Python SDK wrapper
- [ ] TypeScript SDK

### v1.2
- [ ] Dashboard UI (React admin panel)
- [ ] Custom agent registration via API
- [ ] Webhook delivery system
- [ ] Advanced analytics and reporting

### v2.0
- [ ] Learning from decisions (feedback loop)
- [ ] Cross-project insights
- [ ] Managed cloud option (hive.yourdomain.com)
- [ ] Marketplace for community agents/integrations
- [ ] Enterprise features (SSO, audit logs, compliance)

---

## Glossary

| Term | Definition |
|------|------------|
| **Hive** | The AI command center service |
| **HiveBoss** | Main agent class using Claude Agent SDK |
| **Project** | A tenant/instance within Hive |
| **Agent** | Specialized AI for specific tasks |
| **Subagent** | Delegated agent for parallel work |
| **Cycle** | Scheduled autonomous execution loop |
| **Decision** | AI recommendation for action |
| **Task** | Work item (human or AI) |
| **Budget** | Financial constraints tracking |
| **Runway** | Time until budget depleted |
| **Survival Mode** | Restricted operation when low budget |
| **Integration** | External service connection |
| **MCP** | Model Context Protocol (tool interface) |
| **Hook** | Interceptor for controlling agent behavior |
| **Claude Agent SDK** | Anthropic's official agentic framework |

---

## Architecture Decision: Why Claude Agent SDK

We chose the Claude Agent SDK over alternatives (Pydantic AI, LangChain, AutoGPT) for these reasons:

1. **Native Agentic Loop** - The SDK handles plan→act→observe automatically
2. **Built-in Tools** - Bash, Read, Write, WebFetch work out of the box
3. **Subagent Support** - First-class delegation to specialized agents
4. **MCP Integration** - Standard protocol for custom tools
5. **Persistent Sessions** - Context maintained across interactions
6. **Hooks System** - Control agent behavior with interceptors
7. **Claude Optimization** - Purpose-built for Claude's capabilities
8. **Same Patterns as Claude Code** - Battle-tested in production

The SDK provides the foundation for truly autonomous operation while maintaining safety through hooks and approval thresholds.

---

*This specification is a living document. Updates will be made as the system evolves.*

*Version 0.2 - Draft - Jan 2025*
