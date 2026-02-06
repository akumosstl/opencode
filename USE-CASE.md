# Execution Flow

# Prompts


1️⃣ spec-normalizer

```bash

You are using the spec-normalizer agent.

Take the RAW SPEC below and convert it strictly into our official ai-feature-spec format (version 1.0). @.opencode/agents/references/feature_spec-template.md

Rules:
- Do NOT invent missing constraints.
- If information is missing, explicitly write: ⚠ Missing Information
- Separate facts from assumptions.
- Extract business objective clearly.
- Identify functional requirements as FR-1, FR-2, etc.
- Identify missing non-functional requirements.
- Add Open Questions for ambiguities.

RAW SPEC:
"""
Implement a Rest Controller following these specification/feature:

3. Lista de Documentos por Status

`GET /api/documents`
Retorna uma lista paginada de documentos filtrados por status e intervalo de datas.

**Parâmetros de Query:**
| Parâmetro | Tipo | Obrigatório | Descrição |
|-----------|------|----------|-------------|
| `status` | string | Sim | Status do documento: `registered`, `processing`, `critiques`, `errors` |
| `start_date` | string (ISO 8601) | Sim | Data inicial (YYYY-MM-DD) |
| `end_date` | string (ISO 8601) | Não | Data final (YYYY-MM-DD) |
| `page` | integer | Não | Número da página (padrão: 1) |
| `per_page` | integer | Não | Itens por página (padrão: 10, máximo: 100) |
| `search` | string | Não | Buscar por número do documento |

**Exemplo de Requisição:**
http
GET /api/documents?status=errors&start_date=2025-11-26&end_date=2025-11-26&page=1&per_page=10


**Resposta:**
json
{
  "data": [
    {
      "document_type": "Document",
      "document_number": "1234567",
      "load_date": "2025-11-27",
      "status": "error_101",
      "status_label": "Error 101",
      "details": "Invalid 'policyInsurance' field. Value must match specified format.",
      "additional_info": "Returned by registry office: Non-existent account"
    },
    {
      "document_type": "Claim",
      "document_number": "0987812",
      "load_date": "2025-11-26",
      "status": "error_registry",
      "status_label": "Registry Error",
      "details": "Returned by registry office: Non-existent account"
    },
    {
      "document_type": "Document",
      "document_number": "7654321",
      "load_date": "2025-11-26",
      "status": "error_vg",
      "status_label": "VG Error",
      "details": "Umbrella VG cancelled"
    }
  ],
  "meta": {
    "current_page": 1,
    "per_page": 10,
    "total_items": 50,
    "total_pages": 5,
    "start_date": "2025-11-26",
    "end_date": "2025-11-26"
  }
}


```

Change from plan mode to agent and prompt:

```bash
Write the result as .md file at: @.opencode/output/normalize-spec.md

```

2️⃣ feature-planner

```bash
You are using the feature-planner agent.

Based strictly on the normalized ai-feature-spec below:

- Extract domain model
- Define use cases
- Define API contracts refinement
- Identify persistence needs
- Identify architectural boundaries
- Identify risks (performance, concurrency, security)
- Do NOT invent constraints not present in the spec
- Highlight architectural gaps

INPUT SPEC:
"""
<PASTE NORMALIZED SPEC HERE>
"""

```

Change from plan mode to agent and prompt:

```bash
Write the result as .md file at: @.opencode/output/normalize-plan-spec-.md

```

3️⃣ task-breaker

```bash
You are using the task-breaker agent.

Convert the technical plan below into atomic, testable engineering tasks.

Rules:
- Tasks must be small and executable.
- Each task must belong to a specific layer (Domain, Application, Infrastructure, API, Test).
- Each task must contain:
  - Description
  - Acceptance Criteria
  - Technical Notes
- No vague tasks.
- No multi-responsibility tasks.

INPUT PLAN:
"""
<PASTE PLANNER OUTPUT HERE>
"""
```

Change from plan mode to agent and prompt:

```bash
Write the result as .md file at: @.opencode/output/normalize-plan-tasks-spec-.md

```


4️⃣ java-executor


```bash
You are using the java-executor agent.

Implement ONLY the following task.

Rules:
- Respect Clean Architecture.
- Do NOT modify unrelated files.
- Write or update tests if required.
- Follow project standards strictly.
- Output git-style diff only.

TASK:
"""
<PASTE SINGLE TASK HERE>
"""


```