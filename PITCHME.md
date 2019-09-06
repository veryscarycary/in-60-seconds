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
#### The Setup
@snapend

@snap[midpoint text-07]

```typescript
const httpLink = new HttpLink({
  uri: BFF_GRAPHQL_URI,
  credentials: 'include',
});

const cache = new InMemoryCache();

const stateLink = withClientState({
  cache,
  resolvers,
  defaults,
});

export const apolloClient = new ApolloClient({
  link: ApolloLink.from([stateLink, httpLink]),
  cache,
});

export default apolloClient;
```
@snapend

@snap[south text-06]
 * order matters when it comes to the link order
@snapend

---
@snap[north]
##### How do we differentiate between local and server calls?
@snapend

@snap[midpoint fragment]
 Well I'm glad you asked
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

@snap[midpoint span-100 text-07]
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

@snap[north span-100]
 #### Local Resolvers
@snapend

@snap[midpoint span-100 text-07]
```typescript

export const setDetails = (_obj, { name, value }, { cache }) => {
  cache.writeQuery({
    query: DETAILS_QUERY,
    data: {
      campaignDetails: {
        __typename: 'CampaignDetails',
        [name]: value,
      },
    },
  });
  return null;
};

// Resolvers

export const detailsResolvers = {
  setDetails,
  setDetailsBoolean,
  handlePrimaryCTASelection,
  handleSecondaryCTASelection,
  handleCTAData,
  rehydrateDetailsCache,
  handleCampaignDetailsFormValidation,
};

```
---

@snap[north]
##### How do I set the default values?
@snapend

---

@snap[north span-100]
 #### Typical default values setup
@snapend

@snap[midpoint text-07]
```typescript
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
#### You may find yourself doing this if you're working with GQL FE + BE
@snapend

@snap[midpoint text-06]
```
export const mapCampaignDeliveryToGql = campaignDelivery => {
  const {
    selectedAudience,
    checkboxSelections,
    areDistrictGuidelinesConfirmed,
    campaignStartDate,
    numberOfDistributions,
  } = campaignDelivery;

  const campaignDeliverySettings = {
    selectedAudience,
    targetAudiences: checkboxSelections.map(selection =>
      omit(selection, '__typename')
    ),
    numberOfDistributions,
    districtGuidelinesConfirmed: !!areDistrictGuidelinesConfirmed,
  };

  if (campaignStartDate) campaignDeliverySettings.startDate = campaignStartDate;

  return campaignDeliverySettings;
};
```
@snapend

@snap[south text-05 span-50]
@ol
- While \_\_typename is necessary on FE, it is not tolerated on BE
@olend
@snapend

---

@snap[north span-100]
#### Local Schema
@snapend

@snap[midpoint span-100 text-07]
 * Look at local.graphql.js in portal *
 
You can extend the Query and Mutation types and add your own graphql types in this file. The system doesn't seem very strict about this. In fact, if you do not create types for your local cache items, the app will still function as expected.
@snapend

---

@snap[north span-100]
#### Using queries / mutations in React
@snapend

@snap[midpoint span-100 text-07]
 Using the graphql HOC seems to be the most straight-forward and decoupled approach.
@snapend

---

@snap[north span-100 text-07]
```
const deliveryConfig = {
  props: ({
    ownProps,
    data: {
      campaignDelivery: {
        campaignStartDate,
        numberOfDistributions,
        selectedAudience,
        checkboxSelections,
      },
    },
  }) => ({
    ...ownProps,
    campaignStartDate,
    numberOfDistributions,
    selectedAudience,
    checkboxSelections,
  }),
};

export default compose(
  graphql(DETAILS_QUERY, detailsConfig),
  graphql(DELIVERY_QUERY, deliveryConfig),
  graphql(SET_DETAILS_MUTATION, { name: 'setDetails' }),
);
```
@snapend

---
@snap[north span-100 text-07]
```
  render() {
    const {
      setDetails,
      campaignStartDate,
      numberOfDistributions,
      selectedAudience,
      checkboxSelections,
    } = this.props;
    
    <Component onClick={() => {
      setDetails({
        variables: {
          name: 'categoryMain',
          value: (category && category.value) || null,
        },
      });
    }}
```
@snapend

---

@snap[north span-100]
### Pros and Cons
@snapend

@snap[west text-05 span-50]
Pros
@ol
- Similar features from Redux, e.g. caching and offline persistence
- If organized properly, can look very clean and straightforward

@olend
@snapend

@snap[east text-05 span-50]
Cons
@ol
- Very particular about shape  e.g. \_\_typename
- Apollo Dev Tools are buggy/not too insightful
- Error logging is not descriptive/helpful
- Updating store while queries are in-flight can cause errors
@olend
@snapend

