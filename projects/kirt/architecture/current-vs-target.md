# Current State vs. Target State

> Distinguishes between the running Foundation deployment (Forge ecosystem on Contabo) and the planned enterprise deployment (Pellera on Azure). KIRT builds against Foundation's REST API and is infrastructure-agnostic.

## Current State: Forge Ecosystem on Contabo

Foundation v3.0 is deployed and operational. All three milestones shipped (v1 MVP, v2 Hardening, v3 Consumer App Readiness).

### Infrastructure

| Server | Role | Spec | Services |
|--------|------|------|----------|
| CVPS2 | App Plane | 8 vCPU / 24 GB / 400 GB | nginx (host), Django API (gunicorn, 3 workers), Celery Worker (prefork, 4 procs), Dagster Webserver + Daemon, Polaris Catalog |
| CVPS1 | Data Plane | 8 vCPU / 24 GB / 400 GB | Weaviate (12 GB limit), PostgreSQL 16 (3 DBs), Redis 7 (512 MB), Ollama (nomic-embed-text, 6 GB) |

### Network

| Layer | Detail |
|-------|--------|
| Cloudflare DNS | foundation.roybales.com → CVPS2 nginx (A record, SSL) |
| Contabo internal (10.0.0.0/22) | App plane ↔ Data plane. All cross-service traffic. Not internet-routable. |
| Tailscale mesh VPN (100.x.x.x) | Operator access: SSH, Dagster UI (:3000), Django admin. MacBook ↔ CVPS1 ↔ CVPS2. |

### Forge Ecosystem Context

Foundation is one component in the broader Forge ecosystem:

| Component | Purpose |
|-----------|---------|
| **Foundation** | Data platform (ingest, normalize, serve) — this project |
| **Intel** | Account intelligence, GTM strategy, templates |
| **Recon** | Deep research engine |
| **Groundwork** | AI app design and ideation |
| **MCP Server** | 13 tools for AI assistant integration |

Consumer apps talk to Foundation via REST API. They never access Weaviate, Iceberg, or PostgreSQL directly.

## Target State: Pellera Enterprise on Azure

The target moves Foundation into an Azure enterprise environment to serve Pellera's internal tooling, starting with KIRT.

### Confirmed Stack

| Component | Technology |
|-----------|-----------|
| Cloud | Azure |
| Auth | AD/SSO via Graph API |
| Storage | Apache Iceberg / Parquet on Azure Blob Storage |
| Backend | Django (consistent with current) |
| Vector DB | Weaviate (consistent with current) |

### What Changes

| Aspect | Current (Contabo) | Target (Azure) |
|--------|-------------------|----------------|
| Hosting | Contabo VPS (CVPS1 + CVPS2) | Azure cloud services |
| Authentication | Token-based | AD/SSO via Graph API |
| Object storage | Local filesystem / Contabo disks | Azure Blob Storage |
| Iceberg catalog | Polaris Catalog (self-hosted) | TBD — Azure-native or managed Polaris |
| Orchestration | Dagster (self-hosted) | TBD — Dagster Cloud or self-managed |
| Task queue | Celery + Redis (self-hosted) | TBD — Azure-managed or self-hosted |
| Embedding model | Ollama (nomic-embed-text, local) | TBD — Azure OpenAI or self-hosted |
| Database | PostgreSQL 16 (self-hosted) | TBD — Azure Database for PostgreSQL or self-hosted |
| Network | Contabo internal + Tailscale VPN | Azure VNets + AD-integrated access |
| DNS | Cloudflare (roybales.com) | Pellera enterprise domain (TBD) |

### What Stays the Same

| Aspect | Detail |
|--------|--------|
| Backend framework | Django |
| Vector database | Weaviate |
| Storage format | Apache Iceberg / Parquet |
| API-first design | Foundation exposes REST APIs; consumer apps call them |
| Separation of concerns | Foundation = data only; no business logic, no UI |

## KIRT's Position

KIRT is infrastructure-agnostic by design:
- Communicates with Foundation via REST API only
- Auth via AD/SSO (works on both Contabo token-based and Azure AD)
- No direct infrastructure dependencies (no Weaviate, Iceberg, or PostgreSQL access)

**This means:** KIRT V1 can demo on Contabo while Azure migration is planned separately. When Foundation migrates, KIRT's only change is the API base URL.

### KIRT Stack (Greenfield)

| Layer | Technology |
|-------|-----------|
| Frontend | React + Vite + Tailwind + Shadcn |
| Backend | Django, MCP-native |
| Data access | Foundation REST API |

KIRT does not replicate Foundation capabilities. It does not:
- Manage data source connections (Foundation does this)
- Run ingestion pipelines (Foundation does this)
- Store raw documents (Foundation does this)
- Provide vector search infrastructure (Foundation does this)

## Migration Considerations

The migration from Contabo to Azure is part of the roadmap but not yet planned in detail.

### Infrastructure Decisions Needed

- **Compute:** Azure VMs vs. Container Apps vs. AKS for Django + Celery + Dagster
- **Database:** Azure Database for PostgreSQL (Flexible Server) vs. self-managed
- **Object storage:** Azure Blob Storage tier (Hot vs. Cool for Iceberg data)
- **Iceberg catalog:** Azure-native options vs. continuing Polaris
- **Embedding:** Azure OpenAI (text-embedding-3) vs. self-hosted Ollama
- **Redis:** Azure Cache for Redis vs. self-managed

### Migration Sequence (Suggested)

1. Provision Azure environment (VNets, storage accounts, identity)
2. Deploy Foundation backend (Django + PostgreSQL) — validate API parity
3. Migrate Iceberg data (Parquet files to Azure Blob Storage)
4. Configure Weaviate on Azure — re-index vectors if embedding model changes
5. Wire AD/SSO authentication — replace token-based auth
6. Validate all 13 API endpoints against current behavior
7. Cut over DNS from Cloudflare (roybales.com) to Pellera enterprise domain

### Risk Areas

| Risk | Detail |
|------|--------|
| Data normalization | Customer names in 3+ variants across 50+ systems, 36 merged companies, two SF instances. Data quality problem that surfaces during migration. |
| Embedding model change | If model changes (nomic → Azure OpenAI), all vectors must be re-indexed. One-time cost, potentially significant at scale. |
| Dagster orchestration | Scheduling and asset management may need reconfiguration for Azure networking and storage paths. |
| Contabo retention | Determine whether Contabo stays as dev/staging or is decommissioned post-migration. |

### Impact on KIRT

**None for V1.** KIRT builds against the Foundation REST API. Azure migration is a Foundation infrastructure concern. KIRT V1 demos on Contabo, migrates to Azure when Foundation does. The only KIRT-side change is updating the API base URL and switching to Azure AD tokens (which KIRT already targets for auth).
