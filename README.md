# CHIP-2023-08-AuthGuard: AuthGuard Standard

        Title: AuthGuard Standard
        Type: Standards
        Layer: Applications
        Owners: Mathieu Geukens, bitcoincashautist 
        Status: Draft
        Initial Publication Date: 2023-08-09
        Latest Revision Date: 2023-08-09

## Summary

AuthGuard is a standard for authchain management which makes updating the authchain more secure and convenient.

## Deployment

This proposal does not require coordinated deployment.

## Motivation

AuthGuard is a standardized covenant for managing an authchain which tokenizes the authority to issue reserved supply and publish metadata updates. Tokenizing the authority to update the authchain with an AuthKey NFT makes updating the authchain more secure and convenient.

The AuthGuard standard allows the AuthKey NFT to be held in a normal CashTokens wallet without running the risk of accidentally spending the authchain. By tokenizing the authority to update the authchain, there is no need for regular wallets to add specific authHead coin-control or to use a separate wallet just for the authHead UTXO, instead any CashTokens wallet can be used to keep the AuthKey NFT.

The authchain contains both the issuance and the metadata updates, but key rotations of the authority are done by transferring the AuthKey NFT.

## Benefits

### Secure

The AuthGuard standard tokenizes the authority to update the authchain, this NFT is the AuthKey and it can be held in a normal CashTokens wallet without running the risk of the wallet accidentally transferring the authHead.

### Light

The AuthGuard standard allows for light wallets to verify the unissued supply of fungible tokens with the same method as proposed in the BCMR specification for the ongoing issuance of fungible tokens.

### Practical

By keeping the authChain under a coveant and making an AuthKey NFT, the issuer of a token can just keep the Authorithy to issue new supply or update the token's metadata in a CashTokens wallet. 
To actually issue new supply or update the metadata, the issuer can use an application for token creators, connect his wallet with the AuthKey and sign the wanted interaction with the AuthGuard covenant.

## Technical Specification

Tokens and other identities can use following the AuthGuard standard from the Genesis transaction or opt-in afterwards.

### The Genesis Setup

The genesis setup for the AuthGuard standard has the following structure:

- output index 0: AuthGuard output, holding the unreleased tokens
- output index 1: AuthKey NFT
- output index 2: released tokens (optional)
- output index 3: OP_RETURN with self-cerified BCMR info (optional)

The AuthGuard covenant is created as the 0th-index output in the genesis transaction.
The covenant is specified in `AuthGuard.cash`
For a fungible token project, the covenant holds the reserved supply.
The second ouptut is the AuthKey NFT, which is always created alongside the AuthGuard covenant.
For a fungible token project, the AuthKey NFT can have the same tokenId as the fungible tokens.
For a fungible token project, the 3rd output are the tokens released into circulation right away.
The last output is an optional BCMR on-chain metadata publication.

### Opt-in Post-Genesis

Already created identities can still adopt the AuthGuard standard by transferring their authHead to the AuthGuard covenant.
Fungible token projects can send part of the supply to the AuthGuard covenant to mark it as reserved.
Because it is not possible anymore at this later stage to create an AuthKey NFT from the same category as the identity itself, 
a new category should be created for a new AuthKey NFT.

### Metadata

Tokens created with the AuthGuard standard can use a BCMR extension to tell applications that the reserve supply is kept at the authhead.
For identities which adopted the AuthGuard psot-genesis, a `authNft` tokenId needs to be added to BCMR metadata so that the AuthKey NFT can easily be found. 

```
{
  "extensions": {
    "tokenStandard": "AuthGuard"
    "authNft": "8473d94f604de351cdee3030f6c354d36b257861ad8e95bbc0a06fbab2a2f9cf"
  }
}
```

### Tracking the Reserved Supply

Tokens using the AuthGuard standard make an important distinction between genesis supply, total supply, circulating supply and reserved supply.
These concepts are well defined in the [CashTokens specification](https://github.com/cashtokens/cashtokens).
Wallets, blockexplorers and other applications might want to display this information to users.

Tracking the Reserved Supply works by following the authChain like specified in the [BCMR specification](https://github.com/bitjson/chip-bcmr#providing-for-continued-issuance-of-fungible-tokens)

In user-interfaces for a token following the AuthGuard standard, the token's genesis supply may be labeled as **open-ended** when the maxint 
(`9223372036854775807`) is created in the genesis transaction. 
This shows users that there is no hard limit on the amount of tokens without confusing users with this very large number in user interfaces.

### Verifying the Covenant Contract

Verifying the covenant contract is not strictly necessary as the issuer is trusted with the minting baton so could release the whole locked supply anyway.
If applications do want to check this anyway they need to generate P2SH lockingbytecode for a AuthGuard covenant 
for that specific tokenId and check it against actual P2SH lockingbytecode. 
Equivalently the generated p2sh address from the tokenId could be checked against the actual p2sh address.

## Rationale

This section documents design decisions made in this specification.

### Combining Auths

There is only one authChain so the BCMR specification suggests using it both for metadata updates and unissued supply.
An alternative design explored in the [MBC token standard](https://github.com/mr-zwets/MBC-Token-Standard) proposes to use two authchains to separate both authorities.
However, combining the metadata authority with the issuance authority makes it more convenient for token issuers to manage their authorities 
and utilizes the covenant to protect both authorities from accidental transfers.
It is still possible to split the AuthKey while keeping just one authchain by using covenants, this does make the general setup more complex.

### AuthKey NFT Commitment

The commitment of the AuthKey NFT is left unspecified so projects can use a mutable AuthKey NFT to keep state, for example to restrict the amount issued each time period.
This means any NFT with the correct tokenId is seen as a valid AuthKey NFT.
Because of this, projects following the AuthGuard standard which are using NFTs should create an AuthKey NFT of a different category.

## Evaluation of Alternatives

### BCMR spec

The [BCMR specification](https://github.com/bitjson/chip-bcmr#providing-for-continued-issuance-of-fungible-tokens) details how reserved supply of fungible tokens can be verified by light clients by keeping the reserved supply in the identity output and marked by a `mutable` NFT.

Using an authchain to track the reserved/unissued supply is similar to the current proposal, but for light clients it is not clear when they should check the identity output for reserved supply and when there are multiple such reserved supply markers.
That is why the current proposal proposes a `BCMR` extension to tell clients when they can check the reserved supply by following the authchain.

### MBC spec

The [MBC token standard](https://github.com/mr-zwets/MBC-Token-Standard) proposes to separate the BCMR metadata Auth using two separate authchains for the issuance updates and the metadata updates. The standard proposes to use a covenant with accesskey only for the reserved supply. 
Howerver, many times the two authority should be controlled by the issuer and hence it is more convenient to combine them into one AuthKey NFT.
The advantages of MNC standard are that the authorithy to update the metadata could easily be burned and that the issuance history and the metadata update history would be on separate authChains.

## Implementations

The following software is known to support the AuthGuard Standard:

*(pending initial implementations)*

## Feedback & Reviews

## Acknowledgements

This spec is the result of the AuthGuard contract by bitcoincashautist and its further development and implementation by the Paytaca team
in combination with separate work on a 'Minting Baton Covenant'. This standard has been dropped in favor of AuthGuard.

## Changelog
- **v0.1.0 – 2023-08-09**
  - first published draft version

## Copyright

This document is placed in the public domain.
