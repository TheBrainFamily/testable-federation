# testable-federation
Test your Apollo GraphQL Gateway / Federation micro services. With this package you don't have to worry about the whole complexity that comes with joining the GraphQL federated micro services and preparing them for testing. 

Install it with 
```bash 
npm --save-dev testable-federation
```

Example Usage, for the [Federation Demo From Apollo](https://github.com/apollographql/federation-demo).

Demo with the whole repositorium, code examples, and walk-through tutorial coming this weekend! Stay tuned.

```javascript
const { executeGraphql, setupSchema } = require("testable-federation");
const { gql } = require("apollo-server");

const { typeDefs } = require("./schema");
const { resolvers } = require("./resolvers");

const { typeDefs: typeDefsProducts } = require("../products/schema");

const services = [
  { inventory: { typeDefs, resolvers, underTest: true } },
  {
    products: {
      typeDefs: typeDefsProducts
    }
  }
];

beforeAll(() => {
  setupSchema(services);
});

describe("Based on the data from the external service", () => {
  const query = gql`
    {
      topProducts {
        name
        inStock
        shippingEstimate
      }
    }
  `;

  it("should calculate the shipping estimate", async () => {
    const mocks = {
      Product: () => ({
        upc: "1",
        name: "Table",
        weight: 10,
        price: 10,
        elo: "",
        __typename: "Product"
      })
    };

    const result = await executeGraphql({ query, mocks });
    expect(result.data.topProducts[0]).toEqual({
      name: "Table",
      inStock: true,
      shippingEstimate: 5
    });
  });
  it("should set the shippingEstimate at 0 for an expensive item", async () => {
    const mocks = {
      Product: () => ({
        upc: "1",
        name: "Table",
        weight: 10,
        price: 14000,
        elo: "",
        __typename: "Product"
      })
    };

    const result = await executeGraphql({ query, mocks });
    expect(result.data.topProducts[0]).toEqual({
      name: "Table",
      inStock: true,
      shippingEstimate: 0
    });
  });
});

```
