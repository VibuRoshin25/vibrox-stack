# vibrox-core

`vibrox-core` is the public API gateway for the Vibrox Systems Lab.

## Current responsibilities

- Serve browser-safe lab endpoints behind the web reverse proxy.
- Report service health.
- Connect to internal services over gRPC where that demonstrates a useful
  boundary.

## Current endpoint

```http
GET /health
```

## Deliberate exclusions

User CRUD and custom JWT authentication were removed because the public
portfolio has no account requirement. PostgreSQL was removed from the default
runtime for the same reason. Arena will add new endpoints only after its game
state and bot protocol are defined.
