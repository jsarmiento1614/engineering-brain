# Event Driven Architecture

## Concept

Arquitectura basada en eventos asincrónicos donde los componentes se comunican mediante eventos.

## Ventajas

- Desacoplamiento entre servicios
- Escalabilidad horizontal
- Resiliencia ante fallos
- Flexibilidad para agregar nuevos consumidores

## Desventajas

- Complejidad en debugging
- Eventual consistency
- Necesidad de manejo de eventos duplicados

## Casos de uso

- [[glim-login-flow]]
- [[zerovariance-payment-service]]

## Tecnologías comunes

- Kafka
- RabbitMQ
- NATS
- AWS EventBridge
- Azure Service Bus

## Patterns

### Event Sourcing
Almacenar todos los cambios como secuencia de eventos.

### CQRS
Separar comandos (writes) de consultas (reads).

## Related

- [[microservices]]
- [[system-design]]
