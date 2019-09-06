---

@snap[midpoint span-100]
## Apollo Link State
### in Peachjar systems
@snapend

@snap[south-east span-20]
![Logo](assets/pj-logo-header.svg)
@snapend

---

@snap[midpoint span-100]
## What is Apollo Link State?
@snapend

@snap[midpoint fragment text-07]
“In the past, Apollo users stored their application's local data in a separate Redux or MobX store. With apollo-link-state, you no longer have to maintain a second store for local state. You can instead use the Apollo Client cache as your single source of truth that holds all of your local data alongside your remote data. To access or update your local state, you use GraphQL queries and mutations just like you would for data from a server.”
<br />https://www.apollographql.com/docs/link/links/state/
@snapend

---

@snap[north span-100]
### What kind of outputs can we observe?
@snapend

@ol
- Responses to requests
- Externally-stored state (DB, Cache, Event Streams)
- Logs @fa[pepper-hot]
- Metrics @fa[pepper-hot] @fa[pepper-hot]
- Tracing @fa[pepper-hot] @fa[pepper-hot] @fa[pepper-hot]
@olend

---

![Trifecta](https://peter.bourgon.org/img/instrumentation/01.png)

@size[0.6em](https://peter.bourgon.org/blog/2017/02/21/metrics-tracing-and-logging.html)

---

## 1. Responses to requests

---

@snap[north span-100]
### Benefits & Disadvantages of Responses
@snapend

@ul
- Typically the first indication of a problem
- Can be tied into monitoring (alerts)
- May be intentionally masked from requestor (don't show the deatils of internal errors)
- @color[red](**Low resolution**: doesn't give you a lot of insight into how the request ended up in this state.)
@ulend

---

@snap[north span-100]
### Our approach to Responses
@snapend

@ul
- Impose strict structure on API outputs
- Use custom errors with an [error ontology that supports hierarchies](https://peachjar.atlassian.net/wiki/spaces/ENG/pages/45350931/Error+Ontology)
- Auto map errors to protocol (GQL, REST) responses
- @color[orange](TODO: UI interprets errors within context, or<br />falls back to global error handler.)
@ulend

---

@snap[north span-100]
#### Structured Responses - HTTP/REST
@snapend

@ul
- @color[orange](Success responses are weakly defined.)
- Error responses are always JSON using the [hapijs/Boom](https://github.com/hapijs/boom) framework.
- Core automaps:
  - [Route Abstraction](https://github.com/peachjar/peachjar-core/blob/master/src/Interfaces/Framework/HttpApi/Route.ts#L44)
  - [Error Mapper](https://github.com/peachjar/peachjar-core/blob/master/src/Interfaces/Framework/HttpApi/Utils.ts#L13)
- BFF Framework automaps:
  - [Error Mapper](https://github.com/peachjar/bff-framework/blob/master/src/framework/middleware/errorHandler.ts#L138)
@ulend

---
