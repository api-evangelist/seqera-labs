---
name: seqera-labs-launch-pipeline
description: Launch a Nextflow pipeline as a workflow run on the Seqera Platform and monitor it to completion.
api: Seqera Platform API
generated: '2026-07-21'
method: generated
source: openapi/seqera-labs-platform-openapi.yml
operations:
- ListWorkspacesUser
- ListComputeEnvs
- ListPipelines
- DescribePipelineLaunch
- CreateWorkflowLaunch
- DescribeWorkflow
- DescribeWorkflowProgress
- CancelWorkflow
---

# Launch a Seqera Platform pipeline

Automates launching a pipeline as a workflow run and watching it to completion.

## Prerequisites
- Base URL: `https://api.cloud.seqera.io`
- Auth: `Authorization: Bearer <access-token>` (personal access token created in the Seqera UI).
- Most calls are scoped to a workspace via the `workspaceId` query parameter.

## Steps
1. **Find your workspace** — `ListWorkspacesUser` (`GET /user/{userId}/workspaces`) to get the `workspaceId`.
2. **Pick a compute environment** — `ListComputeEnvs` (`GET /compute-envs?workspaceId=...`); note the `computeEnvId` (or the workspace primary CE).
3. **Choose the pipeline** — `ListPipelines` (`GET /pipelines?workspaceId=...`), then `DescribePipelineLaunch` (`GET /pipelines/{pipelineId}/launch`) to fetch the saved launch config (pipeline repo, revision, params).
4. **Launch** — `CreateWorkflowLaunch` (`POST /workflow/launch?workspaceId=...`) with the launch body (computeEnvId, pipeline, revision, params). Response returns the new `workflowId`.
5. **Monitor** — poll `DescribeWorkflowProgress` (`GET /workflow/{workflowId}/progress`) and `DescribeWorkflow` (`GET /workflow/{workflowId}`) until `status` is `SUCCEEDED`, `FAILED`, or `CANCELLED`.
6. **Cancel if needed** — `CancelWorkflow` (`POST /workflow/{workflowId}/cancel`).

## Conventions & error handling
- Pagination on list endpoints: `max` + `offset` (see `conventions/seqera-labs-conventions.yml`).
- Rate limit: 20 requests/second per token — back off on HTTP 429.
- Errors return a JSON `{"message": ...}` envelope (NOT RFC 9457); launch validation returns `{"message","errors"[]}`. See `errors/seqera-labs-problem-types.yml`.
- No idempotency key — do not blindly retry `CreateWorkflowLaunch` on timeout; check via `ListWorkflows` first.
