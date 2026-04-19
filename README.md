# Event-Driven Lab

Proyecto de ejemplo de arquitectura orientada a eventos con Spring Boot y RabbitMQ.

## Que hace este proyecto

El repositorio contiene dos microservicios:

- `producer-service`: expone un endpoint REST para publicar mensajes en RabbitMQ.
- `consumer-service`: escucha la cola de RabbitMQ y procesa/imprime los mensajes recibidos.

Flujo general:

1. Un cliente HTTP llama al productor.
2. El productor publica el mensaje en `messages.exchange` con `messages.routingkey`.
3. RabbitMQ enruta el mensaje a `messages.queue`.
4. El consumidor recibe el mensaje desde la cola y lo registra en logs.

## Tecnologias

- Java 17
- Spring Boot 4
- Spring AMQP (RabbitMQ)
- Gradle
- Docker y Docker Compose

## Estructura

- `producer-service/`: API REST que publica mensajes.
- `consumer-service/`: servicio que consume mensajes.
- `docker-compose.yml`: levanta RabbitMQ + productor + consumidor.

## Opcion 1: Ejecutar con Docker Compose (recomendado)

### Requisitos

- Docker
- Docker Compose
- Acceso a las imagenes publicadas:
	- `richardug/producer-service`
	- `richardug/consumer-service`

### Pasos

Desde la raiz del proyecto:

```bash
docker compose up -d
```

Verifica contenedores:

```bash
docker compose ps
```

Ver logs del consumidor:

```bash
docker compose logs -f consumer
```

### Probar envio de mensaje

El productor queda expuesto en el puerto `8080`:

```bash
curl -X POST "http://localhost:8080/api/messages/send?message=HolaMundo"
```

Respuesta esperada:

```text
Mensaje 'HolaMundo' enviado!
```

En los logs del consumidor deberias ver algo como:

```text
Mensaje recibido: 'HolaMundo'
>>> Mensaje Procesado: HolaMundo
```

### RabbitMQ Management UI

- URL: `http://localhost:15672`
- Usuario: `guest`
- Password: `guest`

### Detener servicios

```bash
docker compose down
```

## Opcion 2: Ejecutar local (sin Docker para los servicios)

Puedes correr RabbitMQ en Docker y los microservicios con Gradle.

### 1) Levantar solo RabbitMQ

```bash
docker compose up -d rabbitmq
```

### 2) Ejecutar productor

```bash
cd producer-service
./gradlew bootRun
```

### 3) Ejecutar consumidor (en otra terminal)

```bash
cd consumer-service
./gradlew bootRun
```

### 4) Probar envio

```bash
curl -X POST "http://localhost:8080/api/messages/send?message=PruebaLocal"
```

## Build y tests

Para compilar y ejecutar pruebas:

Productor:

```bash
cd producer-service
./gradlew clean test build
```

Consumidor:

```bash
cd consumer-service
./gradlew clean test build
```

## Configuracion relevante

En ambos servicios se usa RabbitMQ con estas propiedades principales:

- Host: `rabbitmq` (en Docker Compose)
- Puerto: `5672`
- Usuario: `guest`
- Password: `guest`

Nombres de mensajeria usados por el productor:

- Exchange: `messages.exchange`
- Queue: `messages.queue`
- Routing key: `messages.routingkey`

El consumidor escucha la cola `messages.queue`.

## Endpoint disponible

- `POST /api/messages/send?message=<texto>`

Ejemplo:

```bash
curl -X POST "http://localhost:8080/api/messages/send?message=Evento123"
```

## Notas

- `depends_on` en Compose controla orden de inicio, pero no garantiza que RabbitMQ este listo para conexiones inmediatamente.
- Si el productor/consumidor falla al arrancar por conexion, reinicia el servicio unos segundos despues.