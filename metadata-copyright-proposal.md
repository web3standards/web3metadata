## Simple Summary

A standardized way to retrieve legal information (e.g. copyright, IP, license information) for non-fungible tokens (NFTs) to allow end-users and developers to make informed choices around NFT ownership and usage.

## Motivation

The lack of clear, complete copyright information has led to the following types of challenges:

* [DMCA Takedown requests for unauthorized derivative projects](https://twitter.com/NotLarvaLabs/status/1507141704150880258)
* [Unexpected changes in copyright/IP rights](https://twitter.com/carlypreilly/status/1555676216534917120)
* [Unexpected restrictions of using IP for an asset after years of ownership](https://twitter.com/punk4156/status/1467515691452534793)

With millions of dollars at stake now (and the potential for billions of dollars at stake in the future), these challenges have real consequences for NFT creators and communities as well as the owners, DAOs, and businesses looking to own and/or commercialize them.

Additionally, the lack of clarity around legal rights and usage will stifle innovation. Larger companies and organizations that wish to participate in these new economies will not want to face financial and legal costs as well as damage to their brands for mistakes in compliance.

Fortunately, solutions for these types of issues have been implemented in other domains on the internet (see Creative Commons and media implementations like Flickr). To that end, we can draw from these as well as address any web3 specific needs.

## Painting Done

**NOTICE! This is an early draft, and the following is subject to minor/major changes.**

An ideal outcome of this proposal would be contain most/all of the following qualities:

* Multi-chain (not just Ethereum)
* Consistent format (query and response)
* Data is public (not private or gated by token or payment)
* Data is available (onchain or offchain)
* Append friendly (e.g. before, during, or post mint)
* Changelog history (for changes in ownership or permissions)
* Extendable (to all legal considerations)

A typical query and response might look like the following:


A more technical and complete version of this query and response might look like the following.

`${api-URI}/nft/${chain}/${contract-type}/${token-ID}/legal`

An example using a BAYC NFT.

GET `https://api.web3metadata.com/nft/ethereum/0xbc4ca0eda7647a8ab7c2061c2e118a18a936f13d/2580/legal`

A fictious example using one with explicit copyright information:

And a response might be as follows 

GET `https://api.web3metadata.com/nft/ethereum/0xabcdefeda7647a8ab7c2061c2e118a18a936f13d/100/legal`

```
chainID: 1
contract-type: ERC-721,
contract-address: 0xabcdefeda7647a8ab7c2061c2e118a18a936f13d,
token-ID: 100,
name: Metadata4Ever,
legal:
  copyright:
    license: Attribution 4.0 International (CC BY 4.0),
    license-short: cc-by-4.0
  owners:
    Rick Manelius		
  contact-info:
    ethereum-address: 0x123456678,
    email: copyright-inquiries@manelius.com,
    phone: NA
  IP: NA
  software-license: NA
  trademarks: NA
  changelog:
    2:	
      timestamp: 2022-09-26 06:01:01,
      url: ipfs://ipfs/Qm12343ARVgxv6rXqVPiikMJ8u2NLgmgszg13pYrDKEoiz
    1:
      timestamp: 2022-09-25 06:01:01,
      url: ipfs://ipfs/Qm56783ARVgxv6rXqVPiikMJ8u2NLgmgszg13pYrDKEoiw
```

Please note that `IP`, `software-license`, and `trademarks` keys are examples of where legal information could be extended to include many other things. They could be explicitly claimed by their inclusion and referencing. They could be explicitly excluded by specifying `NA` or implicitly excluded by being undefined alongside other factors.

### What Is NOT Being Proposed

No changes in business logic for ERC-721 or ERC-1155s is being suggested. The focus here is making the copyright (and other legal information) available for onchain and offchain. Specifically:

* Onchain: A schema pattern included within the tokenURI reply.
* Offchain: A schema pattern for all IPLD blocks and/or API responses.

This information could be used by other smart contracts to determine whether or not a given NFT's media could be used for a given purpose. The goal here is not to enforce at the protocol level, but to make this information readily available and consistent at the smart contract, API, and dapp levels.

## Specification

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119. 

### Before Mint

NFT markeplaces and minting tools MAY implement some or all of the copyright factors noted above. The ideal minimums would be: 

* `legal.copyright.license` (default value "All Rights Reserved")
* `legal.copyright.license-short` (default value "all-rights-reserved")

If this if information is being a added at time of mint, a dropdown select option SHOULD appear to give users the ability to select all gradations between "All Rights Reserved" and "Public Domain Work". This list SHOULD mirror Flickr as follows:

* All Rights Reserved
* Public Domain Work
* Public Domain Dedication (CC0)
* Attribution-NonCommercial-ShareAlike (CC BY-NC-SA 2.0)
* Attribution-NonCommercial (CC BY-NC 2.0)
* Attribution-NonCommercial-NoDerivs (CC BY-NC-ND 2.0)
* Attribution (CC BY 2.0)
* Attribution-ShareAlike (CC BY-SA 2.0)
* Attribution-NoDerivs (CC BY-ND 2.0)

Copyright owners may wish to provide their explicit name and contact information attached to a specific asset for commercial use opportunities. To that end, an OPTIONAL step would be to obtain/include the following fields under `legal.copyright`.

* `owners`: An array of strings specifying each partial owner (as a person or legal entity).
* `contact-info`: Onchain and offchain messaging pointers to the copyright owners. This MAY include onchain references (e.g. Ethereum Addresses or ENS profiles) and offchain references (e.g. `email`, `phone`, etc.). 

Notice! Given the fact that contact information would be publicly visible, end-users SHOULD NOT be prompted to use their personal contact information and SHOULD only specificy a contact address that is dedicated for this use only.

While this proposal didn't explicitly recommend this, NFT marketplaces or minting tools MAY include a direct link to the copyright license selected. For example, a user selecting `Attribution-NonCommercial-ShareAlike` MAY want to include a link to [by-nc-sa](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode). This could be included under `legal.copyright` as `license-details`.

### After Mint

While it would be ideal for NFTs to include this information at the individual token or global minting contract level, the reality is that many NFTs have already been minted with [frozen metadata](https://support.opensea.io/hc/en-us/articles/1500012270982-What-is-freezing-metadata-). To that end, we need a strategy whereby the minting entity can sign messages and append data to pre-existing data within tokenURI.

Here we need to address several key factors:

* Authority. Any appended data MUST come from the copyright owners and include signatures.
* Recency. All APIs MUST be able to determine they have the more recent copy of the copyright information.
* Availability. To ensure recency, all data MUST be publicly and accessible.

An approach to this might be as follows:

1. Visit a minting tool w/copyright append capabilities.
2. Connect with wallet address specified by the minting contract OR copyright owners.
3. Enter the token's contract address and token IDs you want to update (if all, leave token ID blank)
4. Enter the updated copyright information.
5. Click to generate transaction with message to sign.
6. Sign transaction in wallet

### Storage

This part of the proposal is important, but could diverge into various options. One that I am most famliar with would be built off the Laconic Network watchers, which would include the following steps:

* Trigger: Offchain event triggers downstream events.
* Conversion/Upload: IPLD block generated and pushed into Filecoin.
* Index: GraphQL endpoint provided for search and serving data.

This solution would ensure permanence, caching, indexing, and serving data. Given the use of IPLD and Filecoin, different technologies could be used on the front end to perform all of these functions.

## Reference

Examples of proposals for other metadata initiatives:

* [EIP-2981: NFT Royalty Standard](https://eips.ethereum.org/EIPS/eip-2981)

Articles (non-comprehensive) that list issues and considerations around copyright:

* [A16z Creates NFT Licensing Framework to Standardize Collectors’ Rights](https://blockworks.co/a16z-creates-nft-licensing-framework-to-standardize-collectors-rights/)
* [Let’s Talk About a Schema.org for NFTs](https://blockworks.co/a16z-creates-nft-licensing-framework-to-standardize-collectors-rights/)

Twitter Threads that list issues and considerations around copyright:

* [NFT copyright info needs to exist on-chain.](https://twitter.com/rickmanelius/status/1524035705990791168)
* [NFT Copyright](https://twitter.com/rickmanelius/status/1567329082135744513)

Please feel free to submit more in the queue below, and I'll update the list above.