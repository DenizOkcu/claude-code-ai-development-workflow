# ADR-002: Local-First Architecture with Docker Milvus

## Status: Accepted
## Date: 2026-03-15

## Context

The user requires the retrieval layer to work locally without cloud dependencies. Zilliz Claude Context supports two storage backends: Zilliz Cloud (managed) and self-hosted Milvus. Milvus Lite (in-process SQLite mode) is Python-only and not supported by the Node.js SDK.

## Decision

Use **Docker Milvus** as the vector store for local-first operation:

```bash
docker run -d --name milvus \
  -p 19530:19530 -p 9091:9091 \
  -v milvus-data:/var/lib/milvus \
  milvusdb/milvus:latest milvus run standalone
```

Combined with **Ollama** for local embeddings, this provides a fully offline retrieval pipeline.

## Alternatives Considered

| Option | Pros | Cons |
|--------|------|------|
| **Docker Milvus (chosen)** | Fully local, persistent, handles BM25 + vector natively | Requires Docker; ~200-500MB RAM overhead |
| **Zilliz Cloud** | Zero local infrastructure; managed scaling | Cloud dependency; API key required; latency; cost |
| **Milvus Lite (SQLite)** | Zero Docker; single file; lightest weight | Not supported in Node.js SDK (Python only) |
| **sqlite-vec + custom server** | Zero Docker; embedded; lightest | Requires building custom MCP server (rejected in ADR-001) |

## Consequences

### Positive
- Fully offline — no internet required after initial setup
- Data stays local — no code leaves the machine
- Persistent storage via Docker volume — survives container restarts
- Single Milvus instance serves all projects

### Negative
- Docker is a prerequisite — users without Docker cannot use retrieval
- ~200-500MB additional RAM usage
- Docker container must be running before Claude Code session starts

### Mitigation
- Setup wizard checks for Docker and Milvus container status
- Skills degrade gracefully if Milvus is unavailable
- Setup wizard offers Zilliz Cloud as an alternative for users who prefer managed infrastructure
- `docker start milvus` is a simple recovery command

## References
- [Milvus standalone Docker guide](https://milvus.io/docs/install_standalone-docker.md)
- [Node.js SDK Milvus Lite gap: milvus-io/milvus-sdk-node#354](https://github.com/milvus-io/milvus-sdk-node/issues/354)
