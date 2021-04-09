---
CIP: 16
Title: NFT Metadata Standard
Authors: Alessandro Konrad <alessandro.konrad@live.de>
Comments-URI:
Status: Draft
Type: Informational
Created: 2021-04-08
Post-History: https://forum.cardano.org/t/cip-nft-metadata-standard/45687
License: CC-BY-4.0
---

## Abstract

This proposal defines an NFT Metadata Standard for Native Tokens.

## Motivation

Tokens on Cardano are a part of the ledger. Unlike on Ethereum, where metadata can be attached to a token through a smart contract, this isn't possible on Cardano because tokens are native and Cardano uses a UTxO ledger, which makes it hard to directly attach metadata to a token.
So the link to the metadata needs to be established differently.
Cardano has the ability to send metadata in a transaction, that's the way we can create a link between a token and the metadata. To make the link unique, the metadata should be appended to the same transaction, where the token forge happens:

> Given a token in a EUTXOma ledger, we can ask “where did this token come from?” Since tokens
> are always created in specific forging operations, we can always trace them back through their
> transaction graph to their origin.

(Section 4.1 in the paper: https://hydra.iohk.io/build/5400786/download/1/eutxoma.pdf)

## Considerations

That being said, we have unique metadata link to a token and can always prove that with 100% certainty. No one else can manipulate the link except if the policy allows it to (<a href="#update">update mechanism</a>).

## Specification

This is the registered `transaction_metadatum_label` value

| transaction_metadatum_label | description  |
| --------------------------- | ------------ |
| 721                         | NFT Metadata |

### Structure

The structure allows for multiple token mints, also with different policies, in a single transaction.

```
{
  "721": {
    [policy_id]: {
      [asset_name]: {
        "name": "<name>",
        "image": "<uri>",
        "description": "<description>"

        "type": "<mime_type>",
        "src": "<uri>"

        <other properties>
      },
      ...
    },
    ...,
    "version":"<version>"
  }
}
```

The <b>image</b> and <b>name</b> property are marked as required. <b>image</b> should be an URI pointing to a resource with mime type image/\* used as thumbnail or as actual link if the NFT is an image (ideally <= 1MB).

The <b>description</b> property is optional.

The <b>type</b> and <b>src</b> properties are optional.

The <b>version</b> property is also optional. If not specified the version is 1.0. Ongoing versions require the <b>version</b> key.

This structure really just defines the basis. New properties and standards can be defined later on for varies uses cases. That's why there is an "other properties" tag.

The retrieval of the metadata should be the same for all however.

### Some examples of how this could be used

Video File
```js
{
  "721": {
    [policy_id]: {
      [asset_name]: {
        "name": "Cat Video",
        "image": "<uri>", // stored on ipfs: https://i.ytimg.com/vi/XyNlqQId-nk/hqdefault.jpg 
        "description": "<description>"
        "type": "video/mp4",
        "src": "<uri>" // stored on ipfs: https://www.youtube.com/watch?v=XyNlqQId-nk
        <other properties>
      },
      ...
    },
    ...,
    "version":"<version>"
  }
}
```

PDF Document
```js
{
  "721": {
    [policy_id]: {
      [asset_name]: {
        "name": "Legal Document",
        "image": "<uri>", // stored on ipfs: thumbnail_of_legal_document.jpg
        "description": "<description>"
        "type": "application/pdf",
        "src": "<uri>" // stored on ipfs: legal_document.pdf
        <other properties>
      },
      ...
    },
    ...,
    "version":"<version>"
  }
}
```

### Retrieve valid metadata for a specific token

As mentioned above this metadata structure allows to have either one token or multiple tokens with also different policies in a single mint transaction. A third party tool can then fetch the token metadata seamlessly. It doesn't matter if the metadata includes just one token or multiple. The proceedure for the third party is always the same:

1. Find the latest mint transaction with the label 721 in the metadata of the specific token
2. Lookup the 721 key
3. Lookup the Policy Id of the token
4. Lookup the Asset name of the token
5. You end up with the correct metadata for the token

### <span id="update">Update metadata link for a specific token</span>

Using the latest mint transaction with the label 721 as valid metadata for a token allows to update the metadata link of this token. As soon as a new mint transaction is occuring including metadata with the label 721, the old metadata is overwritten. This is only possible if the policy allows to mint or burn the same token again.

## References

- Mime type: https://tools.ietf.org/html/rfc6838).
- CIP about reserved labels: https://github.com/cardano-foundation/CIPs/blob/master/CIP-0010/CIP-0010.md
- EIP-721: https://eips.ethereum.org/EIPS/eip-721
- URI: https://tools.ietf.org/html/rfc3986, https://tools.ietf.org/html/rfc2397

## Copyright

This CIP is licensed under CC-BY-4.0
