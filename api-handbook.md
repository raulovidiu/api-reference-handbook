# API Quick Reference (2026 Edition)

Reviewed and revised for current best practices, conventions, and tooling as of June 2026. 

---

## HTTP Verbs

- **GET**: Retrieve a resource. Safe and idempotent.
- **POST**: Create a resource or trigger a non-idempotent action.
- **PUT**: Replace a resource entirely. Idempotent.
- **PATCH**: Partially update a resource. Not guaranteed idempotent unless you design it that way.
- **DELETE**: Remove a resource. Idempotent.
- **HEAD**: Same as GET but returns headers only, no body. Useful for cheap existence/metadata checks.
- **OPTIONS**: Returns the HTTP methods supported for a URL. Also used by browsers for CORS preflight requests.

## HTTP Status Codes

- **1xx — Informational**: e.g. 100 Continue.
- **2xx — Success**: 200 OK, 201 Created, 202 Accepted, 204 No Content.
- **3xx — Redirection**: 301 Moved Permanently, 304 Not Modified.
- **4xx — Client Errors**: 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 422 Unprocessable Entity, 429 Too Many Requests.
- **5xx — Server Errors**: 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout.

**Best practice (2026):** use [RFC 9457 Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457) for structured error bodies (`type`, `title`, `status`, `detail`, `instance`) instead of inventing your own ad-hoc error JSON shape.

## Core API Architectures & Paradigms

- **REST**: Still the dominant paradigm for public and partner-facing APIs.
- **gRPC**: The de facto standard for internal service-to-service (east-west) communication, especially in microservices, real-time, and low-latency systems (trading, gaming, IoT). Built on HTTP/2 and Protocol Buffers.
- **GraphQL**: Common for client-driven data fetching, especially front-end teams that need flexible querying across aggregated resources.
- **Event-driven / asynchronous APIs**: Built around message brokers (Kafka, NATS, SQS) and described with **AsyncAPI**. CloudEvents (a CNCF-graduated spec) is the standard envelope for event metadata, used heavily alongside AsyncAPI.
- **Microservices**: The default architecture for non-trivial systems; usually paired with a service mesh and/or API gateway.
- **Serverless**: Mature, commonly used for event handlers and bursty workloads rather than entire systems.

> **SOA**, standalone **SOAP**, and tools built primarily around them are now niche — relevant mainly in legacy enterprise/financial/government integrations. Don't lead with them in a general API reference; they're a special case, not a default.

## GraphQL Ecosystem

As of 2026, REST still holds the large majority of web API market share, but GraphQL usage has grown substantially among large organizations, so it's worth understanding the current ecosystem, not just the spec.

- **GraphQL Federation**: The standard pattern once a GraphQL API grows beyond a single team/service. Multiple independently-owned **subgraphs** are composed into a single unified **supergraph**, queried through one entry point (the **router/gateway**). This lets teams own their slice of the schema independently instead of maintaining one monolithic GraphQL server.
- **Apollo Federation**: The reference implementation and most widely adopted federation approach (companies like Netflix, Expedia, and Booking run it in production). Key concepts to know: `@key` (defines how an entity is uniquely identified across subgraphs), `@external`/`@requires`/`@provides` (declare cross-subgraph field dependencies), and the **Apollo Router** that performs query planning and execution across subgraphs.
- **The GraphQL Foundation's Composite Schema Working Group**: An active effort (with Apollo, Netflix, Hasura, The Guild, and others) to standardize federation patterns into an official spec rather than leaving it Apollo-proprietary — worth watching if you're committing to federation long-term.
- **Alternatives to Apollo**: GraphQL Mesh / Hive Gateway (open-source, supports Apollo Federation compatibility), GraphQL Yoga, and schema-stitching approaches for teams that want federation-like composition without the Apollo stack specifically.
- **When *not* to federate**: Federation adds real operational complexity (a router/gateway layer, a schema registry, cross-team composition discipline). Meta itself has run a single monolithic GraphQL API since 2012. Start monolithic and federate only once team/service boundaries genuinely demand it — don't adopt it pre-emptively.
- **Security note (current best practice):** subgraphs must never be directly reachable from the public internet — only the router should talk to them. This has become an active area of concern as automated scanning makes it easier to find accidentally-exposed subgraphs; enforce network isolation, disable introspection in production, and put auth/authorization enforcement at the router, not scattered across subgraphs.
- **Schema Registry / validation tooling**: Apollo GraphOS (or equivalents) for schema composition checks, breaking-change detection, and monitoring the federated graph — the GraphQL-world counterpart to OpenAPI linting in REST.

## API Design Patterns

- **Adapter Pattern**: Converts one interface into another that clients expect.
- **Decorator Pattern**: Adds behavior to an object dynamically.
- **Proxy Pattern**: Provides a surrogate to control access to another object — the conceptual basis for API gateways.
- **Circuit Breaker Pattern**: Stops calling a failing downstream dependency to prevent cascading failures. Essential in any microservices/gateway discussion and missing from the original list.
- **Backend for Frontend (BFF)**: A dedicated API layer tailored to a specific client (web, mobile) instead of one-size-fits-all general API.
- **Observer Pattern**: One-to-many notification of state changes — underlies most event-driven/webhook designs.

## API Security

Security is the area that has moved the most since this reference was last written. Treat the following as the current baseline:

- **OAuth 2.1**: The current consolidation of OAuth 2.0 best practices. Mandates PKCE for *all* clients (not just public/mobile ones) and drops the Implicit grant and Resource Owner Password Credentials (ROPC) grant entirely.
- **RFC 9700** (IETF, Jan 2025) — *Best Current Practice for OAuth 2.0 Security*. The baseline checklist for any OAuth implementation in 2026: PKCE everywhere, exact (non-wildcard) redirect URI matching, refresh token rotation with reuse detection, sender-constrained tokens via **DPoP** or **mTLS**, and use of **PAR** (Pushed Authorization Requests) and **JAR** (JWT-secured Authorization Requests) to harden the authorization request itself.
- **OpenID Connect (OIDC)**: Authentication layer on top of OAuth 2.x — unchanged in role, still the standard for federated login.
- **JWT (RFC 7519) / RFC 9068**: JWTs remain standard for access tokens; RFC 9068 defines an interoperable JWT access-token profile so resource servers can validate tokens locally.
- **FAPI 2.0**: Financial-grade API security profile, now being adopted well beyond banking as a stricter OAuth/OIDC baseline for regulated industries.
- **mTLS**: Standard for service-to-service authentication inside a mesh, and increasingly for sender-constraining OAuth tokens.
- **API Keys**: Still fine for simple, low-privilege, server-to-server use cases — never as the sole protection for sensitive data or user-context access.
- **Rate limiting & throttling**: Table stakes, normally enforced at the gateway.
- **CORS**: Unchanged in concept; still required for any browser-based cross-origin API access.

## API Testing & Tooling

The testing landscape has diversified significantly. Postman is no longer the default answer.

- **Postman**: Still has the largest ecosystem (mocking, monitoring, docs, team collaboration) but is now a paid, cloud-first product for team use; its free tier is limited.
- **Bruno**: The leading Git-native, local-first open-source alternative — stores collections as plain text files in your repo, no cloud account required. Good default for teams that want Postman-like UX without cloud lock-in.
- **Insomnia**: Broadest protocol coverage (REST, GraphQL, gRPC, WebSocket, SSE) with both local and cloud modes.
- **Hoppscotch**: Open-source, browser-based, self-hostable; closest drop-in for teams wanting zero install.
- **Thunder Client**: VS Code-native, good for staying in-editor.
- **REST Assured** / **Karate**: Still the standard choices for JVM-based automated API test suites (REST Assured for Java-fluent code, Karate for a Gherkin-style DSL testers can read).
- **k6**, **JMeter**, **Artillery**: Performance/load testing. k6 has become a common modern default; JMeter remains relevant for legacy/enterprise pipelines.
- **Schemathesis**, **Dredd**: Schema-based/contract testing — generate tests directly from your OpenAPI spec rather than hand-writing them.
- **Pact**: The standard tool for consumer-driven contract testing between services.
- **WireMock**: The standard for API mocking/service virtualization in test environments.
- **OWASP ZAP**: Standard open-source tool for automated API security scanning.

**Direction of travel:** the field is shifting from manually maintained Postman-style collections toward **spec-driven testing** — generating and maintaining test suites directly from an OpenAPI definition so tests don't silently drift from the live API.

### Contract testing & schema validation: choosing a strategy

This is worth treating as its own decision, not just a tool to bolt on. The three layers below answer different questions and most mature teams run more than one:

- **Schema validation** (OpenAPI/JSON Schema/GraphQL SDL): answers *"does this payload conform to the documented shape?"* — types, required fields, enums, status codes. Tools: **Spectral** and **Redocly** for linting the spec itself; **Schemathesis** and **Dredd** for generating tests from the spec and validating live responses against it; **Prism** for spec-based mocking. This is the cheapest layer to adopt and gives broad structural coverage with minimal setup — a reasonable default for every API, even non-critical ones.
- **Consumer-driven contract testing** (Pact): answers *"will this specific change break a consumer that actually depends on it?"* — the consumer declares the exact interactions it needs, the provider verifies it can satisfy them, all without spinning up both services together. This is the highest-value layer in a microservices architecture with many service-to-service dependencies, but it requires more setup discipline (provider states, a Pact Broker, `can-i-deploy` checks in CI). The **Pact v4 plugin framework** now covers REST, gRPC, GraphQL, and message-based (Kafka/event) contracts, making it usable across a polyglot service estate, not just HTTP.
- **Integration / end-to-end tests**: still necessary, but should be the smallest, slowest, most expensive layer — reserved for critical user journeys that genuinely span multiple services, not a substitute for the two layers above.

**Practical rule of thumb for 2026:** use OpenAPI-based schema validation everywhere as the floor (cheap, automatic, catches drift). Add consumer-driven Pact contracts specifically for your highest-criticality service boundaries (payments, auth, core data) where a silent breaking change is expensive. Don't reach for full E2E tests to catch what a contract or schema check would catch faster and more reliably.

**Shift-left as the operating principle:** the common thread across current guidance is moving these checks as early as possible — spec linting at PR time, contract/schema validation in pre-merge CI, generated tests from the spec rather than hand-written ones, and mocks/mock servers generated directly from the OpenAPI spec so consumer teams can integrate before the provider implementation is even finished.

> Cut from the original: SoapUI and TestRail as flagship general-purpose tools (SoapUI is now a SOAP/legacy-integration specialist, not a default REST testing tool; TestRail is test-case management, not API testing), plus HttpMaster and Assertible, which have little current adoption or visibility.

## API Development Frameworks

- **Node.js / Express**: Still dominant for fast, lightweight JS/TS APIs. **Fastify** and **NestJS** are now common alternatives worth knowing — NestJS especially for larger, more structured TypeScript codebases.
- **Python**: **FastAPI** has become the default choice for new Python APIs thanks to native OpenAPI generation and async support; Django (with Django REST Framework) and Flask remain common for existing codebases.
- **Spring Boot**: Still the standard for enterprise Java APIs.
- **Go**: Increasingly common for high-performance APIs and infrastructure-adjacent services (many gateways, e.g. Kong, are themselves built on Go/Lua/NGINX).

## API Design, Documentation & Standards

- **OpenAPI (formerly Swagger)**: The dominant specification for describing REST APIs; the practical center of "API-first" design — write the spec, generate docs/mocks/tests/client SDKs from it.
- **AsyncAPI**: The OpenAPI-equivalent for event-driven/async APIs (Kafka, MQTT, WebSockets). Adoption has grown substantially as event-driven architecture has spread.
- **JSON:API**, **HAL**, **OData**: Still in use for hypermedia/standardized JSON conventions, mostly in specific ecosystems (OData especially in Microsoft/enterprise data contexts) rather than as universal defaults.
- **Swagger UI / Redoc**: Standard tools for rendering interactive docs from an OpenAPI spec.
- **HATEOAS**: Conceptually still part of "true REST," but in practice rarely fully implemented outside specific hypermedia-heavy APIs — worth knowing, not worth over-indexing on.

## API Gateways & Management Platforms

This is one of the most active areas in the field. Key players and how they differ:

- **Kong**: The most widely adopted open-source gateway, Kubernetes-native, large plugin ecosystem (Lua core, Go/Python/JS plugin support). Strong default for self-managed, microservices-heavy environments.
- **Apache APISIX**: High-performance, NGINX/etcd-based open-source alternative; very strong raw throughput.
- **Apigee** (Google Cloud): The enterprise choice when monetization, partner governance, and deep analytics matter most — at a real cost premium, and tightly coupled to GCP. Note: Apigee Edge (the older private-cloud product) reached end of life in April 2026 — anyone still on Edge needs a migration plan to Apigee X.
- **AWS API Gateway / Azure API Management**: The default choice when you're already committed to that cloud and want zero-ops convenience.
- **Tyk**: Open-source core with no feature lockout (auth, rate limiting, analytics included free); paid tier needed for dashboard/portal/SSO.
- **Newer "edge-native" entrants** (e.g. Zuplo): Worth knowing about — globally distributed, code-first/TypeScript-programmable gateways aimed at developer experience, increasingly marketed around AI-agent and MCP traffic support.

**2026-specific trend: AI Gateway / MCP Gateway.** Several major gateways (Kong, others) now ship plugins or products specifically for proxying and governing traffic between AI agents and APIs via the **Model Context Protocol (MCP)**. If your APIs may be called by AI agents, factor this into your gateway choice.

## API Performance

- **Caching**, **load balancing**, **CDN edge caching**: unchanged fundamentals.
- **HTTP/2 and HTTP/3**: Now a baseline expectation for connection reuse and multiplexing, not a nice-to-have.
- **Edge computing**: Mainstream for latency-sensitive global APIs.
- **Connection pooling / keep-alive**: A meaningful differentiator between gateway choices at high concurrency.

## API Monitoring & Observability

- **OpenTelemetry**: Now the standard, vendor-neutral instrumentation layer for traces, metrics, and logs — this is the single biggest shift in this category. Instrument with OTel and send data to whichever backend you choose.
- **Datadog**, **New Relic**: Still the leading commercial full-stack observability platforms.
- **Prometheus + Grafana**: The standard open-source metrics/dashboarding stack, especially in Kubernetes environments.

## API Infrastructure

- **Kubernetes**: The standard deployment substrate for API workloads at any real scale.
- **Service mesh — Istio, Linkerd**: For consistent mTLS, traffic policy, and observability across services. Increasingly converging with gateway functionality (gateway-as-mesh-ingress).
- **Docker**: Still the standard for containerization and local dev parity.

## API Governance & Lifecycle

- **API-first design**: Writing the OpenAPI/AsyncAPI spec before implementation, and generating code, docs, mocks, and tests from it. This is now the recommended default workflow, not an optional practice.
- **Versioning**: Still essential — URI versioning (`/v1/`) or header-based versioning are both common; pick one and be consistent.
- **API catalogs**: Larger orgs increasingly need a central catalog/discovery layer across multiple gateways to avoid "API sprawl" and inconsistent security policy across teams.
- **API monetization**: Mainstream enough now that most major gateways (Apigee, Zuplo, others) offer it as a built-in feature rather than a bolt-on.

## Designing for AI Agents (new in 2026)

- Industry surveys show a real gap between API-first adoption and AI-agent readiness — most teams have not yet explicitly designed their APIs for agentic callers.
- Practical implications: clear, machine-readable OpenAPI/AsyncAPI specs matter more than ever (agents rely on them to know what's callable); consider how your auth model handles agent-initiated calls; and watch for excessive/unauthorized call volume from agents as a new class of abuse to rate-limit and monitor for.
- **MCP (Model Context Protocol)** has emerged as the standard way to expose tools/APIs to AI agents. Whether or not you adopt it directly, be aware it's now a factor in gateway selection (see AI Gateway / MCP Gateway above).

## Recommended Reading

- [RFC 9700 — Best Current Practice for OAuth 2.0 Security](https://www.rfc-editor.org/rfc/rfc9700) (IETF)
- [RFC 9457 — Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457) (IETF)
- OpenAPI Specification — official docs at openapis.org
- AsyncAPI Specification — official docs at asyncapi.com
- Pact documentation — official docs at docs.pact.io (consumer-driven contract testing)
- Apollo GraphQL — Federation documentation and best-practices series at apollographql.com
- Martin Fowler — writing on API design and microservices patterns
- Google Cloud / AWS / Azure — vendor API security and gateway best-practice guides (useful for platform-specific detail, read alongside the RFCs above rather than instead of them)
