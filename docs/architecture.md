# Architecture Overview

This document describes the system architecture of HealthSense AI, a
multi-agent healthcare assistant built on n8n.

## Table of Contents

- [System Overview](#system-overview)
- [High-Level Architecture](#high-level-architecture)
- [Agent Interaction Flow](#agent-interaction-flow)
- [Workflow Diagrams](#workflow-diagrams)
- [What Lives Where](#what-lives-where)
- [External Services](#external-services)
- [Data Flow](#data-flow)
- [Trust Boundaries](#trust-boundaries)
- [Design Invariants](#design-invariants)

## System Overview

HealthSense AI is a conversational healthcare assistant that helps users with
three primary tasks:

1. **Doctor Booking**: Find doctors by name or specialty, check availability,
   book appointments, and send confirmation emails.
2. **Diagnostics**: Recommend lab tests based on symptoms and send preparation
   reminders via email.
3. **Hospital Information**: Look up hospitals by location, compare quality
   ratings, and check emergency services and ambulance availability.

The system uses a coordinator pattern: a single user-facing agent (the
Coordinator) receives all chat messages and routes them to specialized
sub-agents based on the user's intent.

## High-Level Architecture

```mermaid
flowchart TB
    User([User via Browser])
    User -->|Chat message| CT[Chat Trigger<br/>Public webhook]

    subgraph Coordinator["Coordinator Agent"]
        CT --> CA[AI Agent<br/>GPT-4o-mini]
        MEM[Buffer Memory<br/>10 messages] -.->|context| CA
    end

    CA -->|doctor/booking query| DBA
    CA -->|symptom/lab test query| DA
    CA -->|hospital/emergency query| HCA

    subgraph DBA["Doctor Booking Agent"]
        DBA_AGENT[AI Agent<br/>GPT-4o]
        DBA_AGENT -->|lookup| DI[doctors_info<br/>Excel/SharePoint]
        DBA_AGENT -->|check slots| DS[doctors_slots<br/>Excel/SharePoint]
        DBA_AGENT -->|send confirmation| EM1[email_user<br/>Outlook]
    end

    subgraph DA["Diagnostics Agent"]
        DA_AGENT[AI Agent<br/>GPT-4o]
        DA_MEM[Buffer Memory] -.->|context| DA_AGENT
        DA_AGENT -->|search tests| LT[Lab Tests<br/>Excel/SharePoint]
        DA_AGENT -->|send reminder| EM2[send_email<br/>Outlook]
    end

    subgraph HCA["Hospital Comparison Agent"]
        HCA_AGENT[AI Agent<br/>GPT-4o]
        HCA_AGENT -->|search hospitals| HGI[Hospital General Info<br/>Airtable]
        HCA_AGENT -->|check emergency| HED[Emergency Data<br/>Excel/SharePoint]
    end

    style Coordinator fill:#e8f5e9,stroke:#2e7d32
    style DBA fill:#e3f2fd,stroke:#1565c0
    style DA fill:#fff3e0,stroke:#e65100
    style HCA fill:#f3e5f5,stroke:#6a1b9a
```

## Agent Interaction Flow

This sequence diagram shows a typical doctor booking conversation from start
to finish:

```mermaid
sequenceDiagram
    actor User
    participant CT as Chat Trigger
    participant Coord as Coordinator<br/>(GPT-4o-mini)
    participant DBA as Doctor Booking<br/>Agent (GPT-4o)
    participant Excel as SharePoint<br/>Excel
    participant Outlook as Microsoft<br/>Outlook

    User->>CT: "I need to see a cardiologist"
    CT->>Coord: Route message
    Coord->>DBA: Tool call with query

    DBA->>Excel: Read doctors_info<br/>(filter: Cardiology)
    Excel-->>DBA: Doctor records
    DBA->>Excel: Read doctors_slots<br/>(filter: available)
    Excel-->>DBA: Available slots

    DBA-->>Coord: "Dr. Smith available<br/>Mar 5 at 9:00 AM.<br/>Confirm?"
    Coord-->>User: Display suggestion

    User->>CT: "Yes, confirm it"
    CT->>Coord: Route reply
    Coord->>DBA: Tool call with<br/>confirmation + context

    DBA-->>Coord: "Please provide<br/>your email"
    Coord-->>User: Ask for email

    User->>CT: "user@example.com"
    CT->>Coord: Route email
    Coord->>DBA: Tool call with email

    DBA->>Outlook: Send confirmation email
    Outlook-->>DBA: Email sent
    DBA-->>Coord: "Appointment confirmed.<br/>Email sent."
    Coord-->>User: Display confirmation
```

## Workflow Diagrams

### Coordinator Agent (Coordinator Agent.json)

```mermaid
flowchart LR
    TRIGGER[Chat Trigger<br/>public webhook] --> COORD[Coordinator<br/>AI Agent]
    LLM1[OpenAI GPT-4o-mini] -.->|language model| COORD
    MEM1[Buffer Memory<br/>10 messages] -.->|memory| COORD
    TOOL_DBA[Doctor Booking<br/>sub-workflow] -.->|tool| COORD
    TOOL_DA[Diagnostics<br/>sub-workflow] -.->|tool| COORD
    TOOL_HCA[Hospital Comparison<br/>sub-workflow] -.->|tool| COORD
```

### Doctor Booking Agent (Doctor Booking Agent.json)

```mermaid
flowchart LR
    TRIGGER2[Execute Workflow<br/>Trigger] --> AGENT2[AI Agent]
    LLM2[OpenAI GPT-4o] -.->|language model| AGENT2
    DI2[doctors_info<br/>Excel Tool] -.->|tool| AGENT2
    DS2[doctors_slots<br/>Excel Tool] -.->|tool| AGENT2
    EMAIL2[email_user<br/>Outlook Tool] -.->|tool| AGENT2
```

### Diagnostics Agent (Diagnostics Agent.json)

```mermaid
flowchart LR
    TRIGGER3[Execute Workflow<br/>Trigger] --> AGENT3[AI Agent]
    LLM3[OpenAI GPT-4o] -.->|language model| AGENT3
    MEM3[Buffer Memory] -.->|memory| AGENT3
    LAB3[Hospital Info<br/>with Lab Tests<br/>Excel Tool] -.->|tool| AGENT3
    EMAIL3[send_email<br/>Outlook Tool] -.->|tool| AGENT3
```

### Hospital Comparison Agent (Hospital Comparison Agent.json)

```mermaid
flowchart LR
    TRIGGER4[Execute Workflow<br/>Trigger] --> AGENT4[AI Agent]
    LLM4[OpenAI GPT-4o] -.->|language model| AGENT4
    AT4[Hospital General Info<br/>Airtable Tool] -.->|tool| AGENT4
    HE4[Hospital Emergency<br/>Data Excel Tool] -.->|tool| AGENT4
```

## What Lives Where

| Responsibility | File | Key Nodes |
|---|---|---|
| User-facing chat interface | `workflows/Coordinator Agent.json` | "When chat message received" (Chat Trigger) |
| Intent routing and orchestration | `workflows/Coordinator Agent.json` | "Coordinator" (AI Agent with system prompt) |
| Conversation memory | `workflows/Coordinator Agent.json` | "Agent Simple Memory" (Buffer Window, 10 messages) |
| Doctor lookup | `workflows/Doctor Booking Agent.json` | "doctors_info" (Microsoft Excel Tool) |
| Slot availability check | `workflows/Doctor Booking Agent.json` | "doctors_slots" (Microsoft Excel Tool) |
| Appointment confirmation email | `workflows/Doctor Booking Agent.json` | "email_user" (Microsoft Outlook Tool) |
| Lab test recommendations | `workflows/Diagnostics Agent.json` | "Hospital Info with Lab Tests" (Microsoft Excel Tool) |
| Test preparation email reminders | `workflows/Diagnostics Agent.json` | "send_email" (Microsoft Outlook Tool) |
| Hospital search and comparison | `workflows/Hospital Comparison Agent.json` | "Hospital General Info" (Airtable Tool) |
| Emergency and ambulance lookup | `workflows/Hospital Comparison Agent.json` | "Hospital Emergency Data" (Microsoft Excel Tool) |

## External Services

| Service | Purpose | Auth Method | Used By |
|---|---|---|---|
| OpenAI API | LLM inference (GPT-4o, GPT-4o-mini) | API Key | All workflows |
| Microsoft SharePoint / OneDrive | Excel data storage for doctors, slots, lab tests, emergency data | OAuth2 | Booking, Diagnostics, Hospital |
| Microsoft Outlook | Sending confirmation and reminder emails | OAuth2 | Booking, Diagnostics |
| Airtable | Hospital general information database (~2,988 records) | Personal Access Token | Hospital Comparison |

## Data Flow

```mermaid
flowchart TD
    subgraph DataSources["Data Sources"]
        SP[SharePoint / OneDrive<br/>Excel Workbooks]
        AT[Airtable<br/>Hospital_General_Information]
    end

    subgraph Datasets["Datasets in SharePoint"]
        D1[doctors_info_data<br/>28 doctors]
        D2[doctors_slots_data<br/>3,024 slots]
        D3[Lab_Tests_Trimmed<br/>3,402 test records]
        D4[hospitals_emergency_data<br/>31 zip codes]
    end

    subgraph AirtableData["Airtable Data"]
        D5[Hospital_General_Information<br/>~2,987 hospitals]
    end

    SP --> D1 & D2 & D3 & D4
    AT --> D5

    D1 & D2 --> DBA_NODE[Doctor Booking Agent]
    D3 --> DA_NODE[Diagnostics Agent]
    D4 & D5 --> HCA_NODE[Hospital Comparison Agent]

    DBA_NODE & DA_NODE --> OL[Microsoft Outlook<br/>Email Delivery]
    OL --> INBOX[User Inbox]
```

## Trust Boundaries

```mermaid
flowchart TB
    subgraph UNTRUSTED["Untrusted (User Input)"]
        UI[Chat Widget<br/>Browser]
    end

    subgraph BOUNDARY["Trust Boundary"]
        CT_B[Chat Trigger<br/>No authentication]
    end

    subgraph TRUSTED["Trusted (Server-Side)"]
        AGENTS[AI Agents<br/>n8n Server]
        CREDS[Credential Store<br/>n8n Encrypted]
    end

    subgraph EXTERNAL["External Services"]
        OPENAI[OpenAI API]
        MS[Microsoft Graph]
        AIR[Airtable API]
    end

    UI -->|user message| CT_B
    CT_B -->|passes to| AGENTS
    AGENTS -->|uses| CREDS
    AGENTS -->|API calls| OPENAI & MS & AIR

    style UNTRUSTED fill:#ffcdd2,stroke:#c62828
    style BOUNDARY fill:#fff9c4,stroke:#f9a825
    style TRUSTED fill:#c8e6c9,stroke:#2e7d32
    style EXTERNAL fill:#bbdefb,stroke:#1565c0
```

Key trust boundary observations:

1. **No webhook authentication**: The Chat Trigger node is set to
   `public: true`. Any user who knows the webhook URL can send messages.
   This is the primary entry point for untrusted input.
2. **User input flows to LLM prompts**: The user's chat message is passed
   directly into the AI Agent's prompt. The system prompts constrain behavior
   but do not sanitize input.
3. **User input reaches email fields**: The user provides the recipient email
   address for confirmation emails. This value is passed to the Outlook tool
   without server-side validation.
4. **Credential isolation**: n8n stores credentials encrypted and separate
   from workflow definitions. The JSON files only contain credential IDs,
   not values.

## Design Invariants

1. **Coordinator never answers directly**: The Coordinator Agent's system
   prompt enforces that it only relays responses from sub-agent tools. It
   never generates healthcare information on its own.
2. **Sub-agents are stateless**: The Doctor Booking and Hospital Comparison
   agents have no memory nodes. The Coordinator must inject prior context
   into each tool call. The Diagnostics Agent has a buffer memory keyed to
   a static session ("latest"), which means concurrent users would share
   context (a known limitation).
3. **Data is read-only**: All workflows read from external data sources but
   never write back. Appointments are "booked" by sending a confirmation
   email, not by updating a database record.
4. **Single LLM provider**: All agents use OpenAI models. The Coordinator
   uses GPT-4o-mini for cost efficiency; sub-agents use GPT-4o for accuracy.
5. **Execution order v1**: All workflows use n8n's v1 execution order, which
   processes nodes breadth-first.
