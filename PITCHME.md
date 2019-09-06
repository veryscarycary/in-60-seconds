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

@snap[west text-05 span-50]
Pros
@ol
- Similar features from Redux, e.g. caching and offline persistence
@olend
@snapend

@snap[east text-05 span-50]
Cons
@ol
- Very particular with call  e.g. \_\_typename
- Apollo Dev Tools are buggy/not too insightful
- Error logging is not descriptive/helpful
- Updating store while queries are in-flight can cause errors
@olend
@snapend

---
@snap[north]
##### Then how do we differentiate between local and server calls?
@snapend

@snap[midpoint fragment]
 I'm glad you asked
@snapend

---
@snap[north span-100]
 #### Typical query
@snapend

@snap[midpoint text-07]
```
import gql from 'graphql-tag';

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
 #### Typical mutation
@snapend

@snap[midpoint text-07]
```
import gql from 'graphql-tag';


export const SET_CHECKBOX_SELECTIONS_MUTATION = gql`
  mutation setCheckboxSelections($checkboxSelections: [Audience]!) {
    setCheckboxSelections(checkboxSelections: $checkboxSelections) @client
  }
`;
```

@snapend

---

@snap[north]
##### How do I set the default values?
@snapend

---

@snap[north span-100]
 #### Typical default values setup
@snapend

@snap[midpoint text-07]
```
export const deliveryDefaults = {
  campaignDelivery: {
    __typename: 'CampaignDelivery',
    selectedAudience: 'parents',
    checkboxSelections: [],
    areDistrictGuidelinesConfirmed: false,
    campaignStartDate: null,
    numberOfDistributions: 1,
    formValidation: {
      __typename: 'CampaignDeliveryFormValidation',
      isFormValid: false,
    },
  },
};
```

---

@snap[north span-100]
#### \_\_typename is very important
@snapend

@snap[midpoint span-100 text-08]
 Your queries/mutations will fail if \_\_typename is not declared in your defaults for an objects and they MAY fail if you do not provide it in your payload to a mutation.
@snapend

---

@snap[north span-100]
#### Local Schema
@snapend

@snap[midpoint span-100 text-07]
 * Look at local.graphql.js in portal *
 
 However, somewhat contrary to the last slide...
 
You can extend the Query and Mutation types and add your own graphql types in this file. The system doesn't seem very strict about this. In fact, if you do not create types for your local cache items, the app will still function as expected.
@snapend

---
