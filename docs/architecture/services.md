# Service Architecture

## Active services

| Service | Interface | Responsibility |
| --- | --- | --- |
| `vibrox-web` | HTTP | Portfolio, Systems Lab UI, and `/api` reverse proxy |
| `vibrox-core` | REST externally, gRPC internally | Public lab API gateway |
| `vibrox-echo` | gRPC | Structured logging experiment |
| `vibrox-dns` | DNS over UDP/TCP | Forwarding resolver experiment |
| `vibrox-arena` | gRPC | Authoritative game state and bot decisions |

## Communication boundary

Browsers communicate only with `vibrox-web`. Nginx forwards `/api/*` to
`vibrox-core`, avoiding cross-origin configuration and preventing direct public
access to internal services. Core translates browser-safe HTTP requests into
internal gRPC calls where an experiment benefits from them.

## Persistence

No active experiment currently requires durable relational persistence, so
PostgreSQL is not part of the default runtime. Add storage only when a concrete
feature—such as match history or aggregate bot analytics—defines its retention
and consistency requirements.

## Identity

The Systems Lab is public and does not currently need accounts. The former
custom auth service is archived. A future private feature should integrate a
maintained OIDC provider instead of introducing custom credential handling.
