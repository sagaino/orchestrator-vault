---
title: "Redis-Backed Refresh Token Rotation with Sliding Expiration Pattern"
type: pattern
tags: [pattern, orchestrator-promotion]
created: 2026-08-27
updated: 2026-08-27
provenance_schema: 1
orchestrator_run: task-016-20260827T084612Z-362b3dbb
sources: ["[[03-Sources/other/orchestrator-runs/task-016-20260827T084612Z-362b3dbb.json]]"]
confidence: 0.95
owner: "local:sagaino"
review_by: 2027-02-23
supersession: ACTIVE
---

# Redis-Backed Refresh Token Rotation with Sliding Expiration Pattern

## Overview

Clean Architecture pattern for Refresh Token Rotation (RTR) and sliding expiration using Redis session snapshots, ensuring single-use token invalidation and credential revocation.

## Purpose

Reusable cross-project backend authentication pattern for Golang/Redis clean architecture, preventing token replay attacks via atomic deletion upon exchange and extending session lifespan dynamically.

## Considerations

- Immediate Invalidation: Old refresh token key must be deleted from Redis upon verification before issuing new token pair to prevent replay attacks.
- Sliding Expiration: Refresh token TTL (e.g., 7 days) is renewed on every rotation cycle while keeping session state synchronized.
- Atomic Logout Revocation: Logout must clear both user session key (session:user:<userId>) and active refresh token (refresh:token:[REDACTED]
- Error Normalization: Expired, missing, or malformed refresh tokens should consistently map to 401 Unauthorized / AccessControlError.

## Related Knowledge

- [[01-Knowledge/patterns/backend/stateful-cache-backed-jwt-session-guard-with-context-manifest-activity-tracking]]
- [[01-Knowledge/patterns/backend/modular-jwt-auth-middleware-with-redis-stateful-session-activity-tracking]]
- [[01-Knowledge/patterns/mobile/dio-interceptor-token-refresh-request-replay-pattern]]

## Source

- [[03-Sources/other/orchestrator-runs/task-016-20260827T084612Z-362b3dbb.json]]
