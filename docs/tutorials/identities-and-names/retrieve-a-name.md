# Retrieve a name

In this tutorial we will retrieve the name created in the [Register a Name for an Identity tutorial](../../tutorials/identities-and-names/register-a-name-for-an-identity.md). Additional details regarding identities can be found in the [Identity description](../../explanations/identity.md).

## Prerequisites
- [General prerequisites](../../tutorials/introduction.md#prerequisites) (Node.js / Dash SDK installed)

## Code

::::{tab-set-code}

```javascript JavaScript - Resolve by Name
//  Resolve by Name
const Dash = require('dash');

const client = new Dash.Client({ network: 'testnet' });

const retrieveName = async () => {
  // Retrieve by full name (e.g., myname.dash)
  return client.platform.names.resolve('<identity name>.dash');
};

retrieveName()
  .then((d) => console.log('Name retrieved:\n', d.getData()))
  .catch((e) => console.error('Something went wrong:\n', e))
  .finally(() => client.disconnect());
```
```javascript JavaScript - Revolve by Record
// Revolve by Record
const Dash = require('dash');

const client = new Dash.Client({ network: 'testnet' });

const retrieveNameByRecord = async () => {
  // Retrieve by a name's identity ID
  return client.platform.names.resolveByRecord(
    'dashUniqueIdentityId',
    '<identity id>',
  );
};

retrieveNameByRecord()
  .then((d) => console.log('Name retrieved:\n', d[0].getData()))
  .catch((e) => console.error('Something went wrong:\n', e))
  .finally(() => client.disconnect());
```
```javascript JavaScript - Search for Name
// Search for Name
const Dash = require('dash');

const client = new Dash.Client({ network: 'testnet' });

const retrieveNameBySearch = async () => {
  // Search for names (e.g. `user*`)
  return client.platform.names.search('user', 'dash');
};

retrieveNameBySearch()
  .then((d) => {
    for (const name of d) {
      console.log('Name retrieved:\n', name.getData());
    }
  })
  .catch((e) => console.error('Something went wrong:\n', e))
  .finally(() => client.disconnect());
```

::::

## Example Name

The following example response shows a retrieved name (`user-9999.dash`):

```json
{
  "label": "Alice",
  "normalizedLabel": "alice",
  "normalizedParentDomainName": "dash",
  "preorderSalt": "lQEiixHMO5TJmbKwKtavg6eAkxuXzvSlrs/OX9glcYI=",
  "records": {
    "dashAliasIdentityId": null,
    "dashUniqueIdentityId": "YhCPn6pSbZ11hCiFmFL6WJkmC3GSwuUSzhS4QAy84EF"
  },  
  "subdomainRules": {
    "allowSubdomains": false
  }
}
``` 

## What's Happening

After we initialize the Client, we request a name. The [code examples](#code) demonstrate the three ways to request a name:

1. Resolve by name. The `platform.names.resolve` method takes a single argument: a fully-qualified name (e.g., `user-9999.dash`). 
2. Resolve by record. The `platform.names.resolveByRecord` method takes two arguments: the record type (e.g., `dashUniqueIdentityId`) and the record value to resolve. 
3. Search. The `platform.names.search` method takes two arguments: the leading characters of the name to search for and the domain to search (e.g., `dash` for names in the `*.dash` domain). The search will return names that begin the with string provided in the first parameter.

After the name is retrieved, it is displayed on the console.