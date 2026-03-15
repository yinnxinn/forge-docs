# AI Forge API

The AI Forge API powers the natural language → DSL generation workflow.

## Forge Session Lifecycle

```
1. Create Session → 2. Send Prompts → 3. Review Artifacts → 4. Validate → 5. Publish
```

## Create Session

```
POST /api/v1/forge/sessions
Authorization: Bearer {token}
Content-Type: application/json

{
  "title": "Build a CRM"
}
```

## Send Prompt

```
POST /api/v1/forge/sessions/{session_id}/prompt
Authorization: Bearer {token}
Content-Type: application/json

{
  "content": "Create a customer management app with fields: company name, contact person, email, phone, status (Lead/Prospect/Customer/Churned)"
}
```

The backend:
1. Builds context (existing apps, user role)
2. Constructs a system prompt with DSL specification
3. Sends to LLM (OpenAI-compatible API)
4. Extracts DSL from the response
5. Validates via Pydantic
6. Saves as a Forge artifact

**Response:**
```json
{
  "message": {
    "role": "assistant",
    "content": "I've created a Customer Management app..."
  },
  "artifact": {
    "id": 1,
    "type": "dsl",
    "content": { ... },
    "status": "draft"
  }
}
```

## List Artifacts

```
GET /api/v1/forge/sessions/{session_id}/artifacts
Authorization: Bearer {token}
```

## Validate DSL

```
POST /api/v1/forge/validate
Authorization: Bearer {token}
Content-Type: application/json

{
  "dsl": { ... }
}
```

Returns validation result (ok/errors).

## Publish (Register App)

```
POST /api/v1/forge/sessions/{session_id}/artifacts/{artifact_id}/publish
Authorization: Bearer {token}
```

Equivalent to calling `/apps/register` with the artifact's DSL. Creates the app, registers metadata, and mounts the menu.

## How AI Generation Works

The Forge uses a structured prompt that includes:

1. **System prompt**: DSL specification, field types, examples
2. **User context**: Existing apps, tenant info, role
3. **User prompt**: Natural language description

The LLM generates a JSON DSL that is:
- Extracted from the response (handles markdown code blocks)
- Validated by `LingtarnDSL` Pydantic model
- Stored as a Forge artifact
- Available for preview (read-only) before publishing

## Conversation History

Each session maintains a conversation history, allowing iterative refinement:

```
User: "Create a task management app"
AI: [generates DSL v1]
User: "Add a priority field and due date"
AI: [generates DSL v2 with additions]
User: "Add a workflow: Open → In Progress → Done"
AI: [generates DSL v3 with workflow]
```
