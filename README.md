# Solid Development eXperience (SDX) SDK

This library is meant to facilitate easy development of Solid applications. The library uses a (SDL) GraphQL Schema as input and generates a client proxy sdk from it, that can be imported and used to interact with data on a Solid Pod. 

This enables a couple of features:

* The generated schema can be used by any of your favorite GraphQL IDE plugins.
* You can write your own GraphQL queries against data hosted on a targeted Solid Pod.
* Your IDE's plugins can leverage the GraphQL Schema to generate IntelliSense suggestions when writing GraphQL Queries.
* Your IDE's plugins can leverage the type information in the `sdk.generated.ts` file to generate IntelliSense suggestions while writing TypeScript code.
* A fully typed SolidClient generated from your queries, can be used in your application

*This library ideally works in tandem with the SDX cli library (@solidlab/solid-sdx-cli)*

## Install

```bash
npm install @solidlab/solid-sdx-ts
```

## Usage

### Init sdx project
```bash
# Setup project with sdx cli and then install a shape package.

#

```


This triggers the following steps:

    * Once the SHACL is downloaded, it is put in the `/src/.sdx-gen/shacl` folder. 
    * The GraphQL schema will be generated and written to `/src/.sdx-gen/graphql/schema.graphqls`.
    * You can write queries and mutations in your `/src/gql` folder.
    * This the queries and schema will be used to generate the `/src/sdx-gen/sdk.generated.ts` file.
    * To create a SolidClient, import this file in your code.

### Instantiate SolidClient

```ts
import { SolidLDPBackend, SolidLDPContext, StaticTargetResolver } from '@solidlab/sdx-sdk';
import { getSolidClient, Sdk } from 'src/.sdx-gen/sdk.generated';

// Create a backend that statically resolves to one (Pod) URI.
const resolver = new StaticTargetResolver('http://localhost:3000/complex.ttl'); 
const defaultContext = new SolidLDPContext(resolver);
const { requester } = new SolidLDPBackend({ defaultContext });
// Create the client with fully types support
const client: Sdk = getSolidClient(requester);
```

### Using the client

After running `sdx generate sdk`, given these sample queries: 

**Queries example**
```graphql
query listContacts {
    contactCollection {
        id
        givenName
        familyName
    }
}

query getContact($id: String!) {
    contact(id: $id) {
        id
        givenName
        familyName
        address {
            streetLine
            postalCode
            city
            country
        }
    }
}

mutation createContact($id: ID, $givenName: String!, $familyName: String!) {
    createContact(input: {id: $id, givenName: $givenName, familyName: $familyName}) {
        id
        givenName
        familyName
    }
}
```

The client can be used like this:

```ts
import { SolidLDPBackend, SolidLDPContext } from '@solidlab/sdx-sdk';
import { Contact, getSolidClient, Sdk } from 'src/.sdx-gen/sdk.generated';

// Create a backend that statically resolves to one (Pod) URI.
const resolver = new StaticTargetResolver('http://localhost:3000/complex.ttl'); 
const defaultContext = new SolidLDPContext(resolver);
const { requester } = new SolidLDPBackend({ defaultContext });
// Create the client with fully types support
const client: Sdk = getSolidClient(requester);

// Use the client to read
const contacts = (await client.listContacts()).data;
contacts.forEach({givenName, familyName} => console.log(`${givenName} ${familyName}`));

const contact = (await client.getContact('http://example.org/cont/tdupont')).data;
console.log(`${contact.givenName} ${contact.familyName}`);

// Use the client to write
await client.createContact({
    id: 'http://example.org/cont/jdoe',
    givenName: 'John',
    familyName: 'Doe'
});
```

## Configuration


**.sdxconfig**

*This is actually configuration of the cli library, but some options here will generate a different Sdk format.*

```json
{
    "formatVersion": "1.0.0",   // Format version for backwards compatibility and migration
    "catalogs": [               // Catalogs to contact for searching and installing
        {
            "name": "SolidLab Catalog",
            "uri": "https://catalog.solidlab.be"
        }
    ],
    "options": {
        "autoGenerate": true,   // Autogenerate again after each shape (un)install
        "resultEnvelope": false // If true:  wrap in an `ExecutionResult` envelope with `data` and `error` keys.
                                // If false: output only `data` value.
    }
}

```
