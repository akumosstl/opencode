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

# AI Feature Spec (Version 1.0)

## 1. Business Objective

Enable users to retrieve and monitor document submissions by providing a filtered, paginated view of documents based on their processing status and submission date range.

- **Why does this feature exist?** To allow users (likely internal operations/claims staff) to query and track document submissions that require attention (e.g., errors, critiques) or are in various processing states.

- **What problem does it solve?** Provides visibility into document pipeline status, enabling operators to identify and address documents that failed processing or need review.

- **What metric does it improve?** Document processing throughput, error resolution time.

---

## 2. Functional Requirements

**FR-1:** The endpoint `GET /api/documents` MUST return a paginated list of documents filtered by `status`.

**FR-2:** The endpoint MUST support filtering by `status` with the following valid values: `registered`, `processing`, `critiques`, `errors`.

**FR-3:** The endpoint MUST require a `start_date` parameter in ISO 8601 format (YYYY-MM-DD).

**FR-4:** The endpoint MUST support an optional `end_date` parameter in ISO 8601 format (YYYY-MM-DD).

**FR-5:** The endpoint MUST return pagination metadata including: `current_page`, `per_page`, `total_items`, `total_pages`.

**FR-6:** The endpoint MUST accept optional `page` parameter (default: 1).

**FR-7:** The endpoint MUST accept optional `per_page` parameter (default: 10, maximum: 100).

**FR-8:** The endpoint MUST support optional `search` parameter to filter by document number.

**FR-9:** Each document in the response MUST include: `document_type`, `document_number`, `load_date`, `status`, `status_label`, `details`, `additional_info`.

---

## 3. Non-Functional Requirements

⚠ **Missing Information:**
- Performance constraints (response time SLA)
- Availability requirements
- Throughput expectations (RPS)
- Concurrency limits
- Rate limiting rules

**Assumptions (pending confirmation):**
- Response time should be under 500ms for p95
- System should handle at least 100 RPS

---

## 4. Domain Model Impact

**Entities Involved:**
- Document entity (core)
- DocumentStatus enum

**New entities?** No

**Modified entities?** No (read-only operation)

**Invariants?** None identified

---

## 5. API Contracts

### Endpoint
**Method:** GET
**Path:** `/api/documents`

### Request
```
GET /api/documents?status=errors&start_date=2025-11-26&end_date=2025-11-26&page=1&per_page=10
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | Yes | Document status: `registered`, `processing`, `critiques`, `errors` |
| `start_date` | string | Yes | ISO 8601 date (YYYY-MM-DD) |
| `end_date` | string | No | ISO 8601 date (YYYY-MM-DD) |
| `page` | integer | No | Page number (default: 1) |
| `per_page` | integer | No | Items per page (default: 10, max: 100) |
| `search` | string | No | Document number search |

### Response
```json
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

### Error Cases
⚠ **Missing Information:**
- 400: Invalid parameters (not defined in spec)
- 401: Unauthorized (not defined in spec)
- 500: Internal server error (not defined in spec)

**Assumption:** Standard HTTP error codes apply per REST conventions.

---

## 6. Data & Persistence

⚠ **Missing Information:**
- Tables affected by this query
- Required indexes for filtering
- Transaction boundaries (read-only)

**Assumptions:**
- Query reads from PostgreSQL/SQL Server databases mentioned in project
- Pagination uses LIMIT/OFFSET or cursor-based approach

---

## 7. Edge Cases

⚠ **Missing Information:**
- Duplicate requests handling
- Race conditions (not applicable for read-only)
- Partial failures
- Network retries

**Identified edge cases:**
- `end_date` earlier than `start_date` (validation needed)
- `per_page` exceeding maximum (validation needed)
- `status` with invalid value (validation needed)
- `start_date` with invalid format (validation needed)
- Empty result set
- Date range with no documents

---

## 8. Observability

⚠ **Missing Information:**
- Logs required
- Metrics required
- Tracing required

**Suggested:**
- Log incoming requests with parameters (excluding sensitive data)
- Track query execution time metric
- Record filter combinations for usage analytics

---

## 9. Security Considerations

⚠ **Missing Information:**
- Required roles/permissions
- Authentication mechanism
- Authorization rules

**Assumptions:**
- Endpoint requires authentication (JWT or session)
- Users can only see documents they are authorized to access
- Input validation for all query parameters required

---

## 10. Definition of Done

- [ ] Unit tests for filtering logic
- [ ] Integration tests for endpoint
- [ ] API documented (OpenAPI/Swagger)
- [ ] Pagination validated
- [ ] Error handling tested
- [ ] Performance tested (if SLA defined)
- [ ] Code reviewed

---

## Open Questions

1. What is the maximum date range allowed (e.g., can users query 1 year of data)?
Not informed yet, but we can assume a reasonable limit (e.g., 1 year) to prevent performance issues.

2. Should the `search` parameter support partial matches or only exact matches?
A: Partial matches would be more user-friendly, allowing users to find documents even if they only remember part of the document number.

3. Are there authorization rules limiting which documents a user can see?
A: Not informed yet, but we can assume users can only see documents they are authorized to access based on their roles/permissions.

4. What is the expected response time SLA?
A: Not informed yet, but we can assume under 500ms for p95 based on typical internal API performance requirements.

5. Should the response include error details for non-error statuses?
A: Not for all, even some error statuses may not have it. Depends on the error and what information is available from the registry office.

6. Is there rate limiting required for this endpoint?
A: Not informed yet, but we can assume standard rate limiting (e.g., 100 RPS) for internal APIs.

7. Should `additional_info` field be present for all statuses or only errors? 
A: Not for all, even some error statuses may not have it. Depends on the error and what information is available from the registry office.




Change from plan mode to agent and prompt:

```bash
Write the result as .md file at: @.opencode/output/normalize-plan-spec-.md

```

3️⃣ task-breaker


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

# Document Submission Query Feature - Architectural Analysis

## 1. Domain Model

### Core Entities
- **Document** (read-only, aggregate root)
  - document_type: String
  - document_number: String (searchable)
  - load_date: LocalDate
  - status: DocumentStatus enum
  - status_label: String
  - details: String (nullable)
  - additional_info: String (nullable)

### Enums
- **DocumentStatus**: `registered`, `processing`, `critiques`, `errors`

### Value Objects
- DateRange: start_date (required), end_date (optional)
- Pagination: page, per_page, offsets

### Invariants
- start_date must be ≤ end_date when both provided
- per_page ≤ 100
- status must be valid enum value

---

## 2. Use Cases

### Primary Use Cases
| ID | Use Case | Actor | Preconditions |
|----|----------|-------|---------------|
| UC-1 | Filter documents by processing status | Operations staff | Authenticated, valid date range |
| UC-2 | Query documents within date range | Operations staff | Authenticated, valid start_date |
| UC-3 | Search documents by document number | Operations staff | Authenticated, partial or exact match |
| UC-4 | Paginate through large result sets | Operations staff | Authenticated, valid pagination params |

### Secondary Use Cases
- UC-5: Export filtered document list for reporting
- UC-6: Monitor document queue backlog by status counts

---

## 3. API Contracts Refinement

### Request Contract (Current → Refined)

| Parameter | Type | Required | Validation | Default |
|-----------|------|----------|------------|---------|
| status | String | Yes | Must match enum values | N/A |
| start_date | String | Yes | ISO 8601 (YYYY-MM-DD), ≤ end_date | N/A |
| end_date | String | No | ISO 8601 (YYYY-MM-DD), ≥ start_date | start_date + 1 day |
| page | Integer | No | ≥ 1 | 1 |
| per_page | Integer | No | 1-100 | 10 |
| search | String | No | Max length, alphanumeric | N/A |

### Response Contract (Refined)

```json
{
  "data": [
    {
      "document_type": "string",
      "document_number": "string",
      "load_date": "YYYY-MM-DD",
      "status": "registered|processing|critiques|errors",
      "status_label": "string",
      "details": "string|null",
      "additional_info": "string|null"
    }
  ],
  "meta": {
    "current_page": "integer",
    "per_page": "integer",
    "total_items": "integer",
    "total_pages": "integer",
    "start_date": "YYYY-MM-DD",
    "end_date": "YYYY-MM-DD|null"
  },
  "links": {
    "first": "string|null",
    "prev": "string|null",
    "next": "string|null",
    "last": "string|null"
  }
}
```

### Error Response Standardization
```json
{
  "error": {
    "code": "VALIDATION_ERROR|BUSINESS_RULE_VIOLATION|UNAUTHORIZED|FORBIDDEN|NOT_FOUND|INTERNAL_ERROR",
    "message": "Human-readable description",
    "details": [
      {
        "field": "parameter_name",
        "message": "Specific validation failure"
      }
    ],
    "timestamp": "ISO-8601 timestamp",
    "path": "/api/documents"
  }
}
```

---

## 4. Persistence Needs

### Database Targets
- Primary: SQL Server (ODS database per existing architecture)
- Secondary: PostgreSQL (if document metadata distributed)

### Query Pattern
```sql
SELECT 
    document_type,
    document_number,
    load_date,
    status,
    status_label,
    details,
    additional_info
FROM documents
WHERE status = :status
  AND load_date BETWEEN :start_date AND COALESCE(:end_date, :start_date)
  AND (:search IS NULL OR document_number LIKE CONCAT(:search, '%'))
ORDER BY load_date DESC
OFFSET :offset ROWS
FETCH NEXT :per_page ROWS ONLY
```

### Transaction Boundary
- **Propagation**: REQUIRED (read-only transaction)
- **Isolation**: READ_COMMITTED
- **Timeout**: 30 seconds

---

## 5. Architectural Boundaries

### Service Layer Boundaries
```
┌─────────────────────────────────────────────────────────┐
│                    API Gateway / Controller             │
│                  GET /api/documents                     │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                   DocumentQueryService                  │
│  - validateRequest(status, start_date, end_date)        │
│  - executeQuery(filter, pagination, search)             │
│  - enrichResults(documents)                              │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                DocumentRepository                       │
│  - findByStatusAndDateRange(spec)                       │
│  - countByStatusAndDateRange(spec)                      │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│                 Database / JPA                          │
│                 SQL Server / PostgreSQL                 │
└─────────────────────────────────────────────────────────┘
```

### Cross-Cutting Concerns
- **Security**: Authentication filter → Authorization check → Document filter by user scope
- **Observability**: Request tracing, metrics, logging (sanitized)
- **Validation**: DTO validation → Business rule validation
- **Caching**: Consider Redis cache for frequent queries (status + date combination)

### Integration Points
- Authentication service (JWT validation)
- Authorization service (document access policy)
- Metrics collector (Prometheus)
- Logging infrastructure

---

## 6. Risks

### Performance Risks
| Risk | Severity | Mitigation |
|------|----------|------------|
| Full table scan on large date ranges | High | Index on (status, load_date), limit date range |
| N+1 query on result enrichment | Medium | Use JOIN or fetch all related data upfront |
| Slow LIKE queries on document_number | Medium | Use FULLTEXT index or trigram similarity |
| High offset values causing timeout | Medium | Implement cursor-based pagination for deep pages |

### Concurrency Risks
| Risk | Severity | Mitigation |
|------|----------|------------|
| Concurrent requests with different filters | Low | Stateless service, read-only transactions |
| Pagination inconsistencies under updates | Low | Consider eventual consistency for large datasets |

### Security Risks
| Risk | Severity | Mitigation |
|------|----------|------------|
| Information disclosure via unauthorized access | Critical | Row-level security, user document scoping |
| Denial of service via expensive queries | Medium | Rate limiting, max date range, query timeout |
| Sensitive data in logs (document numbers) | Medium | Request logging with field masking |

### Data Risks
| Risk | Severity | Mitigation |
|------|----------|------------|
| Timezone confusion on date filtering | Medium | Normalize all dates to UTC, clarify client timezone |
| Date range spanning database boundaries | Low | Multi-database query federation (if needed) |

---

## 7. Architectural Gaps

### Critical Gaps (Require Clarification)
1. **Authorization Model**: No specification of how document visibility is scoped to users/roles
2. **Date Range Limit**: Maximum permissible range not defined (1 year? 90 days?)
3. **Search Behavior**: Partial vs exact match for document_number not confirmed
4. **Response Time SLA**: Performance constraint pending confirmation

### Significant Gaps (Should Address)
1. **Caching Strategy**: No caching mechanism specified for repeated queries
2. **Rate Limiting**: No RPS limits or throttling rules defined
3. **Pagination Approach**: OFFSET-based vs cursor-based not specified
4. **Multi-Tenancy**: Document scoping by tenant/client not addressed

### Minor Gaps (Can Design)
1. **Field Filtering**: API doesn't support response field projection (reduce payload)
2. **Bulk Export**: No endpoint for large result set export (async job)
3. **Analytics**: No endpoint for aggregated counts by status
4. **Webhooks**: No notification mechanism for status changes

---

## 8. Recommendations

### Immediate Actions
1. **Confirm authorization model** with product owner
A: Authorization is doing by other component a bff so no worries about it here
2. **Define date range maximum** to prevent expensive queries
A: one year maximum range
3. **Specify search behavior** (partial vs exact match)
A: partial match to be more user friendly
4. **Validate DocumentStatus enum** against existing status codes
A: use as the maximum possible existing enum values

### Design Decisions Required
1. Pagination strategy: OFFSET (simpler) vs cursor (better for large datasets)
A: Cursor based pagination for better performance on large datasets
2. Cache invalidation policy for document metadata
A: Cache for 5 minutes with TTL
3. Whether to implement async export for large result sets
A: No, not in the scope of this task
4. Logging verbosity level (debug vs info for query parameters)
A: Info level with sensitive data masked


Change from plan mode to agent and prompt:

```bash

Write the result as .md file at: @.opencode/output/normalize-plan-tasks-spec-.md


```


4️⃣ java-executor


You are using the java-executor agent.

Implement ONLY the following task.

Rules:
- Respect Clean Architecture.
- Do NOT modify unrelated files.
- Write or update tests if required.
- Follow project standards strictly.
- Output git-style diff only.

TASK:

### **Task domain-2: Create Document value object**
- **Description**: Implement immutable Document value object with validation
- **Acceptance Criteria**: All fields validated, null safety, equals/hashCode implemented
- **Technical Notes**: Use final fields, validation annotations, constructor validation

