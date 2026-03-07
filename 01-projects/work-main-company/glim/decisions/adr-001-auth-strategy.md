# ADR-001: Authentication Strategy

## Context

Necesitamos implementar un sistema de autenticación seguro para GLIM.

## Decision

Implementar autenticación basada en JWT (JSON Web Tokens).

## Consequences

### Positive
- Stateless: no requiere almacenamiento en servidor
- Escalable: funciona bien con microservicios
- Estándar de la industria

### Negative
- Tokens no pueden ser revocados fácilmente
- Requiere implementar refresh tokens para mejor seguridad

## Alternatives Considered

- Session-based auth: descartado por requerir almacenamiento compartido
- OAuth2: considerado pero demasiado complejo para el caso de uso actual

## Status

- [x] Proposed
- [x] Accepted
- [ ] Deprecated
- [ ] Superseded

## Related

- [[login-flow]]
- [[user-endpoints]]
