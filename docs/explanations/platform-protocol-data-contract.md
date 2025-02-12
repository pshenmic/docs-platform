# Data Contract

## Overview

As described briefly in the [Dash Platform Protocol explanation](../explanations/platform-protocol.md#data-contract), Dash Platform uses data contracts to define the schema (structure) of data it stores. Therefore, an application must first register a data contract before using the platform to store its data. Then, when the application attempts to store or change data, the request will only succeed if the new data matches the data contract's schema.

The first two data contracts are the [DashPay wallet](https://www.dash.org/dashpay/) and [Dash Platform Name Service (DPNS)](../explanations/dpns.md). The concept of the social, username-based DashPay wallet served as the catalyst for development of the platform, with DPNS providing the mechanism to support usernames.

## Details

### Ownership

Data contracts are owned by the [identity](../explanations/identity.md) that registers them. Each identity may be used to create multiple data contracts and data contract updates can only be made using the identity that owns it.

### Structure

Each data contract must define several fields. When using the [JavaScript implementation](https://github.com/dashevo/platform/tree/master/packages/js-dpp) of the Dash Platform Protocol, some of these fields are automatically set to a default value and do not have to be explicitly provided. These include:

* The platform protocol schema it uses (default: defined by [js-dpp](https://github.com/dashevo/platform/blob/master/packages/js-dpp/lib/dataContract/DataContract.js#L352))
* A contract ID (generated from a hash of the data contract's owner identity plus some entropy)
* One or more documents

In the [example contract](#example-contract) shown below, a `contact` document and a `profile` document are defined. Each of these documents then defines the properties and indices it requires.

### Registration

Once a [Dash Platform Protocol](../explanations/platform-protocol.md) compliant data contract has been defined, it may be registered on the platform. Registration is completed by submitting a state transition containing the data contract to [DAPI](../explanations/dapi.md).

The drawing below illustrates the steps an application developer follows to complete registration.

```{eval-rst}
.. figure:: ../../img/data-contract.svg
   :class: no-scaled-link
   :align: center
   :width: 80%
   :alt: Data Contract Registration

   Data Contract Registration
```

### Updates

#### Contract revision history

Dash Platform v0.25 added optional contract revision history storage. Contracts using this feature maintain a record of contract revisions which can be retrieved and verified as needed.

#### Identity key binding

Dash Platform v0.25 added key access rules that enable adding an encryption or decryption identity key that can only be used for the specific contract (or document) designated when the key is added. This provides a more granular and secure approach to key management.

#### Contract updates

Dash Platform v0.22 added the ability to update existing data contracts in certain backwards-compatible ways. This includes adding new documents, adding new optional properties to existing documents, and adding non-unique indices for properties added in the update.

> 📘
>
> For more detailed information, see the [Platform Protocol Reference - Data Contract](../protocol-ref/data-contract.md) page.

## Example Contract

An example contract for [DashPay](https://github.com/dashevo/platform/blob/master/packages/dashpay-contract/schema/dashpay.schema.json) is included below:

```json
{
  "profile": {
    "type": "object",
    "indices": [
      {
        "name": "ownerId",
        "properties": [
          {
            "$ownerId": "asc"
          }
        ],
        "unique": true
      },
      {
        "name": "ownerIdAndUpdatedAt",
        "properties": [
          {
            "$ownerId": "asc"
          },
          {
            "$updatedAt": "asc"
          }
        ]
      }
    ],
    "properties": {
      "avatarUrl": {
        "type": "string",
        "format": "uri",
        "maxLength": 2048
      },
      "avatarHash": {
        "type": "array",
        "byteArray": true,
        "minItems": 32,
        "maxItems": 32,
        "description": "SHA256 hash of the bytes of the image specified by avatarUrl"
      },
      "avatarFingerprint": {
        "type": "array",
        "byteArray": true,
        "minItems": 8,
        "maxItems": 8,
        "description": "dHash the image specified by avatarUrl"
      },
      "publicMessage": {
        "type": "string",
        "maxLength": 140
      },
      "displayName": {
        "type": "string",
        "maxLength": 25
      }
    },
    "required": [
      "$createdAt",
      "$updatedAt"
    ],
    "additionalProperties": false
  },
  "contactInfo": {
    "type": "object",
    "indices": [
      {
        "name": "ownerIdAndKeys",
        "properties": [
          {
            "$ownerId": "asc"
          },
          {
            "rootEncryptionKeyIndex": "asc"
          },
          {
            "derivationEncryptionKeyIndex": "asc"
          }
        ],
        "unique": true
      },
      {
        "name": "ownerIdAndUpdatedAt",
        "properties": [
          {
            "$ownerId": "asc"
          },
          {
            "$updatedAt": "asc"
          }
        ]
      }
    ],
    "properties": {
      "encToUserId": {
        "type": "array",
        "byteArray": true,
        "minItems": 32,
        "maxItems": 32
      },
      "rootEncryptionKeyIndex": {
        "type": "integer",
        "minimum": 0
      },
      "derivationEncryptionKeyIndex": {
        "type": "integer",
        "minimum": 0
      },
      "privateData": {
        "type": "array",
        "byteArray": true,
        "minItems": 48,
        "maxItems": 2048,
        "description": "This is the encrypted values of aliasName + note + displayHidden encoded as an array in cbor"
      }
    },
    "required": [
      "$createdAt",
      "$updatedAt",
      "encToUserId",
      "privateData",
      "rootEncryptionKeyIndex",
      "derivationEncryptionKeyIndex"
    ],
    "additionalProperties": false
  },
  "contactRequest": {
    "requiresIdentityEncryptionBoundedKey": 2,
    "requiresIdentityDecryptionBoundedKey": 2,
    "type": "object",
    "indices": [
      {
        "name": "ownerIdUserIdAndAccountRef",
        "properties": [
          {
            "$ownerId": "asc"
          },
          {
            "toUserId": "asc"
          },
          {
            "accountReference": "asc"
          }
        ],
        "unique": true
      },
      {
        "name": "ownerIdUserId",
        "properties": [
          {
            "$ownerId": "asc"
          },
          {
            "toUserId": "asc"
          }
        ]
      },
      {
        "name": "userIdCreatedAt",
        "properties": [
          {
            "toUserId": "asc"
          },
          {
            "$createdAt": "asc"
          }
        ]
      },
      {
        "name": "ownerIdCreatedAt",
        "properties": [
          {
            "$ownerId": "asc"
          },
          {
            "$createdAt": "asc"
          }
        ]
      }
    ],
    "properties": {
      "toUserId": {
        "type": "array",
        "byteArray": true,
        "minItems": 32,
        "maxItems": 32,
        "contentMediaType": "application/x.dash.dpp.identifier"
      },
      "encryptedPublicKey": {
        "type": "array",
        "byteArray": true,
        "minItems": 96,
        "maxItems": 96
      },
      "senderKeyIndex": {
        "type": "integer",
        "minimum": 0
      },
      "recipientKeyIndex": {
        "type": "integer",
        "minimum": 0
      },
      "accountReference": {
        "type": "integer",
        "minimum": 0
      },
      "encryptedAccountLabel": {
        "type": "array",
        "byteArray": true,
        "minItems": 48,
        "maxItems": 80
      },
      "autoAcceptProof": {
        "type": "array",
        "byteArray": true,
        "minItems": 38,
        "maxItems": 102
      },
      "coreHeightCreatedAt": {
        "type": "integer",
        "minimum": 1
      }
    },
    "required": [
      "$createdAt",
      "toUserId",
      "encryptedPublicKey",
      "senderKeyIndex",
      "recipientKeyIndex",
      "accountReference"
    ],
    "additionalProperties": false
  }
}
```

This is a visualization of the JSON data contract as UML class diagram for better understanding of the structure:

```{eval-rst}
.. figure:: ./img/dashpay-uml.png
   :class: no-scaled-link
   :align: center
   :width: 90%
   :alt: Dashpay Contract Diagram

   Dashpay Contract Diagram
```

View [a full-size copy of this diagram](./img/dashpay-uml.png).
