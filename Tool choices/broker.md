# Broker design: <-- Task to design by Mark & Sebastiaan
- Protocol definition by protobuf

# Protocol
## Transport Protocol
Options:
- ZeroMQ (probably only point-to-point)
- AMQP
- MQTT
- RabittMQ native
  - Evaluate on:
      - exactly-once delivery guarantee want life cycle berichtjes op basis van message ids
      - low latency
      - topics or similar to denote which subject a subscriber has interest in.
      - Many-to-many
      - RBAC
      - HA or persistency with self-healing
      - Retrieve latest value
      - Request, response, async compatible (full-duplex)
      - Beschikbaar voor python, Java, Rust & typescript (liever nog meer)
      - ESDL's kunnen 1GB groot worden, kunnen ze overweg met 1GB?

## Application protocol format
Options:
- Protobuf
- Messagepack
- Json
- https://capnproto.org/

Evaluate on: 
- Forwards compatability
- Backwards compatibility
- Add versioning
- Allow for timestamps, floats with precision, maps, lists
- Optional fields
- Available for Python, Rust, Java & typescript (and preferably more languages)

### Messages
From frontend to backend
- Start workflow1
  - + Response
- Start workflow2
  - + Response
- Start workflow3
  - + Response
- Cancel workflow
  - + Response
- Retrieve default for workflow (while providing ESDL)
    - Elk request/column wordt een repeatable veld.
  - + Response
- Retrieve key figures for workflow (while providing ESDL)
    - Elk request/column wordt een repeatable veld.
- + Response
- Retrieve workflow status e.g. current execution state (waiting etc), (expected start time), (expected end time)

From backend to frontend
- Finish workflow
- Progress
- Workflow Failed (including logging (max 5MB) & exit status)
