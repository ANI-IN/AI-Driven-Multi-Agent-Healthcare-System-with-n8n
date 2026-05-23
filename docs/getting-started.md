# Getting Started

This guide walks you through importing the HealthSense AI workflows into n8n
and running your first conversation.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Option A: n8n Cloud](#option-a-n8n-cloud)
- [Option B: Self-Hosted with Docker](#option-b-self-hosted-with-docker)
- [Importing the Workflows](#importing-the-workflows)
- [Configuring Credentials](#configuring-credentials)
- [Uploading Datasets](#uploading-datasets)
- [Activating the System](#activating-the-system)
- [Running Your First Conversation](#running-your-first-conversation)
- [Verifying Everything Works](#verifying-everything-works)

## Prerequisites

Before you begin, make sure you have:

| Requirement | Details |
|---|---|
| n8n | Version 1.70 or later (Cloud or self-hosted) |
| OpenAI API key | With access to GPT-4o and GPT-4o-mini |
| Microsoft Azure AD app | With Microsoft Graph and Mail.Send permissions |
| Microsoft 365 account | With access to SharePoint / OneDrive and Outlook |
| Airtable account | With a personal access token |
| Browser | Chrome, Firefox, Safari, or Edge (latest version) |

## Option A: n8n Cloud

1. Sign up or log in at [app.n8n.cloud](https://app.n8n.cloud).
2. You will get a managed n8n instance with a public URL for webhooks.
3. Skip to [Importing the Workflows](#importing-the-workflows).

## Option B: Self-Hosted with Docker

1. Make sure Docker is installed and running on your machine.

2. Create a directory for n8n data:
   ```bash
   mkdir -p ~/.n8n
   ```

3. Copy the environment template and fill in your values:
   ```bash
   cp .env.example .env
   # Edit .env with your credentials
   ```

4. Start n8n:
   ```bash
   docker run -it --rm \
     --name n8n \
     -p 5678:5678 \
     --env-file .env \
     -v ~/.n8n:/home/node/.n8n \
     docker.n8n.io/n8nio/n8n
   ```

5. Open your browser and go to `http://localhost:5678`.

6. Create an owner account when prompted.

## Importing the Workflows

Import the workflows in this order (sub-agents first, then the coordinator):

1. **Hospital Comparison Agent**
   - In the n8n UI, click the three-dot menu in the top-right corner.
   - Select "Import from File".
   - Choose `workflows/Hospital Comparison Agent.json`.

2. **Doctor Booking Agent**
   - Repeat the import steps with `workflows/Doctor Booking Agent.json`.

3. **Diagnostics Agent**
   - Repeat the import steps with `workflows/Diagnostics Agent.json`.

4. **Coordinator Agent**
   - Repeat the import steps with `workflows/Coordinator Agent.json`.
   - After import, open the Coordinator workflow and update the three
     sub-workflow tool nodes to point to the correct workflow IDs:
     - Click "Doctor Booking Agent" node, update the workflow reference.
     - Click "Diagnostics Agent" node, update the workflow reference.
     - Click "Hospital Comparison Agent" node, update the workflow reference.

## Configuring Credentials

You need to set up four credential types in n8n. Go to
**Settings > Credentials > Add Credential** for each:

### 1. OpenAI API

- **Type**: OpenAI
- **API Key**: Your OpenAI API key
- **Used by**: All four workflows

### 2. Microsoft Excel (OAuth2)

- **Type**: Microsoft Excel OAuth2 API
- **Client ID**: From your Azure AD app registration
- **Client Secret**: From your Azure AD app registration
- **Required permissions**: Files.Read, Files.ReadWrite
- **Used by**: Doctor Booking Agent, Diagnostics Agent, Hospital Comparison
  Agent

### 3. Microsoft Outlook (OAuth2)

- **Type**: Microsoft Outlook OAuth2 API
- **Client ID**: From your Azure AD app registration
- **Client Secret**: From your Azure AD app registration
- **Required permissions**: Mail.Send
- **Used by**: Doctor Booking Agent, Diagnostics Agent

### 4. Airtable

- **Type**: Airtable Personal Access Token
- **Token**: Your Airtable personal access token
- **Required scopes**: data.records:read
- **Used by**: Hospital Comparison Agent

After creating each credential, open the corresponding workflow nodes and
select the credential you just created.

## Uploading Datasets

The project includes sample datasets in the `datasets/` folder. You need to
upload these to the services the workflows read from:

### SharePoint / OneDrive

Upload the following files to your SharePoint or OneDrive:

| File | Purpose | Used by |
|---|---|---|
| `doctors_info_data.xlsx` | Doctor names, specializations, contacts | Doctor Booking Agent |
| `doctors_slots_data.xlsx` | Appointment time slots and availability | Doctor Booking Agent |
| `Lab_Tests_Trimmed.xlsx` | Lab tests, hospitals, preparation info | Diagnostics Agent |
| `hospitals_emergency_data.xlsx` | Emergency services by zip code | Hospital Comparison Agent |

After uploading, update the Microsoft Excel Tool nodes in each workflow to
point to the new file locations.

### Airtable

1. Create a new Airtable base named `Hospital_General_Information`.
2. Import `Hospital_General_Info_Trim.xlsx` into the base.
3. Note the base ID and table ID from the Airtable URL.
4. Update the Airtable Tool node in the Hospital Comparison Agent workflow.

## Activating the System

1. Open the **Coordinator Agent** workflow.
2. Click the **Active** toggle in the top-right corner to activate it.
3. The sub-agent workflows do not need to be activated separately; they are
   called as tools by the Coordinator.

## Running Your First Conversation

Once the Coordinator Agent is active:

1. Open the Chat Trigger's webhook URL in your browser. You will find it by
   clicking the "When chat message received" node and looking at the
   "Webhook URLs" section (use the Production URL).

2. You should see a styled chat widget with the greeting:
   "Hi there! I am HealthSense AI. How can I assist you today?"

3. Try these sample queries:

   **Doctor Booking**:
   ```
   I need to see a cardiologist
   ```

   **Diagnostics**:
   ```
   What tests should I take for persistent headaches?
   ```

   **Hospital Information**:
   ```
   List hospitals in Alabama
   ```

## Verifying Everything Works

Run through this checklist to confirm the system is operational:

- [ ] The chat widget loads and displays the welcome message
- [ ] A doctor booking query returns doctor names and available slots
- [ ] A diagnostics query returns lab test recommendations with preparation
      instructions
- [ ] A hospital query returns hospital details from the database
- [ ] Confirmation emails arrive in the recipient's inbox
- [ ] The Coordinator correctly routes different query types to the right
      sub-agent

If any step fails, check the n8n execution log for the workflow. Common issues
and fixes are listed in the [Troubleshooting](../README.md#troubleshooting)
section of the README.
