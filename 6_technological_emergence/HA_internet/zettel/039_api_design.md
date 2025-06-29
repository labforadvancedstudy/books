# API Design

## Core Insight
APIs are contracts between programs - formal agreements about how software should talk, enabling machines to cooperate without understanding each other's internals.

Application Programming Interface. The name is perfect. It's literally the face programs present to each other. Like human faces, APIs hide complexity behind simple expressions. "Give me user data" not "query the database, join three tables, format as JSON."

REST conquered because it mapped to HTTP verbs. GET retrieves, POST creates, PUT updates, DELETE removes. This simplicity let any language talk to any service. The web became programmable. Every site potentially an API.

But APIs are promises that break. Version 1 works perfectly. Version 2 "improves" things, breaks everything. Backward compatibility is a chain - each version must support all previous promises. Eventually the chain gets too heavy. APIs deprecate. Code dies.

GraphQL challenged REST - why fetch fixed data shapes when you can query exactly what you need? But flexibility breeds complexity. Simple REST calls became complex GraphQL queries. The pendulum swings between simplicity and power.

APIs reveal organizational philosophy. Open APIs invite innovation. Closed APIs maintain control. Rate limits encode business models. Authentication schemes expose trust models. Show me your API, I'll show you your soul.

## Connections
→ [[024_http_protocol]]
→ [[030_server_client]]
→ [[034_javascript]]
← [[036_distributed_systems]]
← [[048_microservices]]

---
Level: L3
Date: 2025-06-23
Tags: #api #programming #interfaces #contracts