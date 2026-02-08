# Execution Flow

# Prompts


1️⃣ spec-normalizer


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


Refactor the DocumentQueryController (Document Service) locate at:
@sro-submission-document-service-v3\src\main\java\br\com\somosadd\submission\document\controller\DocumentQueryController.java

Refactor DocumentQueryController and all its flow (dtos, services, entities) knowing that the entities, repositories and dtos are locate from sro-submission-entity (JPA Entities library).

Refactor methods like convertToDocumentData in @sro-submission-document-service-v3\src\main\java\br\com\somosadd\submission\document\service\DocumentQueryServiceImpl.java

Guidelines:

To understand the database logic take a look at
@sro-submission-document-service-v3\.opencode\context\project-context.md in the sql/jpa sections.

Special attention at JPA: don´t create any new Enities these are already created at @monitoramento\sro-submission-entity moduel/project

It is important understand that the project has two Entities Managers they are configured in classes at
```@sro-submission-document-service-v3\src\main\java\br\com\somosadd\submission\document\config``` package.

So the queries should be or native or making references to correct schemas.




---



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
