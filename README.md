# API Quick Reference (2026 Edition)

A concise, no-fluff reference covering modern API concepts, standards, security practices, and tooling — reviewed and kept current as of **July 2026**.

This started as a community-shared cheat sheet and has been reviewed and expanded with a focus on what's actually relevant in practice today: current OAuth security guidance (RFC 9700 / OAuth 2.1), the real API testing and gateway landscape, GraphQL Federation, contract testing strategy, and a look at how AI agents are starting to change API design.

## Why this exists

Most "API cheat sheets" floating around online accumulate cruft over time — legacy HTTP verbs nobody uses, tools that were popular five years ago, and lists that read the same whether they were written in 2018 or this year. This repo is an attempt to keep one reference genuinely current, with notes on *why* something was kept, cut, or added, so it's useful as a learning resource and not just a list to memorize.

## What's inside

- **HTTP fundamentals** — verbs, status codes, structured error responses (RFC 9457)
- **Core architectures** — REST, gRPC, GraphQL, event-driven/async APIs, and when each fits
- **GraphQL ecosystem** — Federation, Apollo, subgraphs/supergraphs, and when *not* to federate
- **API design patterns** — including patterns missing from most cheat sheets, like Circuit Breaker and BFF
- **Security** — OAuth 2.1, RFC 9700, PKCE, DPoP/mTLS, FAPI 2.0
- **Testing strategy** — current tooling (Bruno, Insomnia, k6, Pact, Schemathesis, WireMock) plus a dedicated breakdown of schema validation vs. consumer-driven contract testing vs. end-to-end tests, and when to use which
- **Gateways & infrastructure** — Kong, Apigee, APISIX, Tyk, service mesh, and the emerging AI Gateway / MCP Gateway category
- **Documentation & standards** — OpenAPI, AsyncAPI, and what's fallen out of common use (RAML, API Blueprint)
- **Designing for AI agents** — a new, genuinely 2026 concern as agentic callers become a real share of API traffic

## Who this is for

Developers, QA/automation engineers, and anyone studying for API-related domain or just wanting a single page to refresh their understanding of where the ecosystem actually stands today, rather than five years ago.

## A note on currency

API tooling and standards move fast. This reference reflects the state of the field as of **July 2026** and includes inline notes explaining what was changed and why, so it stays useful as a *learning artifact* even as parts of it eventually age. Contributions and corrections are welcome — open an issue or a PR if something here is out of date or you think a section deserves expanding.

## License

This content is shared freely for learning purposes. Feel free to use, adapt, and share it.

(Suggested: license this repo under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) if you want to allow free reuse with attribution, or [MIT](https://opensource.org/licenses/MIT) if you'd rather treat it like a typical open-source text/docs project. Add a `LICENSE` file with your choice before publishing.)

## Contents

- [`API_Quick_Reference_2026.md`](./API_Quick_Reference_2026.md) — the main reference document
