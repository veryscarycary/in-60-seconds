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

---

@snap[midpoint text-07]
“In the past, Apollo users stored their application's local data in a separate Redux or MobX store. With apollo-link-state, you no longer have to maintain a second store for local state. You can instead use the Apollo Client cache as your single source of truth that holds all of your local data alongside your remote data. To access or update your local state, you use GraphQL queries and mutations just like you would for data from a server.”
<br />https://www.apollographql.com/docs/link/links/state/
@snapend

---

@snap[north span-100]
### Pros and Cons
@snapend

@snap[west span-50]
Pros
@ol
- Similar features from Redux, e.g. caching and offline persistence
@olend
@snapend

@snap[east span-50]
Cons
@ol
- Very particular with call  e.g. __typename
- Apollo Dev Tools are buggy/not too insightful
- Error logging is not descriptive/helpful
- Updating store while queries are in-flight can cause errors
@olend
@snapend

---

### Then how do we differentiate between local and server calls?

@snap[midpoint fragment]
 I'm glad you asked
@snapend

---

## Typical query

@snap[midpoint]
```typescript
import gql from 'graphql-tag';
/* eslint-disable graphql/template-strings */
const CAMPAIGN_DETAILS_VALIDATION_CLIENT_QUERY = gql`
  query campaignDetailsValidation {
    campaignDetails @client {
      formValidation {
        __typename
        isFormValid
      }
    }
  }
`;

export default CAMPAIGN_DETAILS_VALIDATION_CLIENT_QUERY;
```

@snapend

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
