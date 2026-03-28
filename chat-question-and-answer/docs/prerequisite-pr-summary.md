# Prerequisite PR-Style Summary

## Goal
Prepare deployment and integration boundaries for future GraphRAG work without changing current query semantics.

## Scope
- Optional graph-stack deployment scaffold in Docker Compose.
- Helm parity for graph-ready backend configuration.
- Non-regression unit tests for existing `/chat` behavior.
- Clear before/after documentation for mentor review.

## Files Changed
- `docker-compose.yaml`
- `chart/values.yaml`
- `chart/templates/egaisample-deployment.yaml`
- `chart/templates/graph-neo4j.yaml` (new)
- `tests/unit_tests/test_server.py`
- `docs/prerequisite-pr-summary.md` (new)

## Before / After

### 1) Deployment Scaffold
Before:
- No optional Neo4j service in `docker-compose.yaml`.
- Backend had no graph-related environment variables in Compose.

After:
- Added optional Neo4j service under `GRAPH` profile.
- Added graph-ready backend env variables (`CHATQNA_PIPELINE_MODE`, `AGENTIC_NEO4J_*`) with safe defaults.
- Baseline startup remains unchanged when `GRAPH` profile is not enabled.

### 2) Helm Parity
Before:
- Helm values did not include graph settings.
- Backend deployment template did not pass graph-ready env variables.
- No optional Neo4j workload template.

After:
- Added `graph.*` settings in `chart/values.yaml`.
- Added graph-ready env vars in `chart/templates/egaisample-deployment.yaml`.
- Added `chart/templates/graph-neo4j.yaml` gated by `graph.enabled`.

### 3) Non-Regression Tests
Before:
- Unit tests covered basic chat success and a small set of error paths.

After:
- Added validation tests for token limit and whitespace-only question.
- Added compatibility test ensuring unknown future payload fields do not break current `/chat` path.

## Key Diff Snippets

### `docker-compose.yaml`
```diff
+      - CHATQNA_PIPELINE_MODE=${CHATQNA_PIPELINE_MODE:-rag}
+      - AGENTIC_NEO4J_URI=${AGENTIC_NEO4J_URI:-bolt://neo4j:7687}
+      - AGENTIC_NEO4J_USER=${AGENTIC_NEO4J_USER:-neo4j}
+      - AGENTIC_NEO4J_PASSWORD=${AGENTIC_NEO4J_PASSWORD:-graphRAG}
+
+  neo4j:
+    image: neo4j:5.26
+    profiles:
+      - GRAPH
```

### `chart/templates/egaisample-deployment.yaml`
```diff
+            - name: CHATQNA_PIPELINE_MODE
+              value: "{{ .Values.Chatqna.env.CHATQNA_PIPELINE_MODE }}"
+            - name: AGENTIC_NEO4J_URI
+              value: "{{ .Values.Chatqna.env.AGENTIC_NEO4J_URI }}"
+            - name: AGENTIC_NEO4J_USER
+              value: "{{ .Values.Chatqna.env.AGENTIC_NEO4J_USER }}"
+            - name: AGENTIC_NEO4J_PASSWORD
+              value: "{{ .Values.Chatqna.env.AGENTIC_NEO4J_PASSWORD }}"
```

### `tests/unit_tests/test_server.py`
```diff
+def test_query_chain_max_tokens_validation(test_client):
+    ...
+
+def test_query_chain_whitespace_question_validation(test_client):
+    ...
+
+def test_query_chain_ignores_future_mode_field(test_client, mocker):
+    ...
```

## Validation

### Unit Tests
Run from `sample-applications/chat-question-and-answer`:
```bash
pytest tests/unit_tests/test_server.py -q
```

### Compose (Baseline)
```bash
docker compose up -d
```

### Compose (Graph Scaffold Enabled)
```bash
docker compose --profile GRAPH up -d
```

### Helm Render (Baseline)
```bash
helm template chatqna ./chart
```

### Helm Render (Graph Enabled)
```bash
helm template chatqna ./chart --set graph.enabled=true
```

## Non-Goals
- No GraphRAG retrieval logic.
- No fused orchestration.
- No answer-generation behavior changes.
- No claim that summer milestones are already implemented.
-This branch intentionally avoids retrieval, orchestration, and answer-generation changes reserved for the summer milestones