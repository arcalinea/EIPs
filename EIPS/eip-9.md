### Title

      Title: Add precompiled BLAKE2b contract
      Author: Tjaden Hess & Jay Graber
      Status: Draft
      Type: Standard Track
      Layer: Consensus (hard-fork)
      Created 2016-06-30

## Abstract

This EIP introduces a new precompiled contract which implements the BLAKE2b cryptographic hashing algorithm.

## Motivation
BLAKE2b is a useful addition to Ethereum's cryptographic primitives. BLAKE2b is faster and more efficient than the SHA hash functions, making it a good complement or alternative to SHA-3. The BLAKE2b algorithm is highly optimized for 64-bit CPUs, and is faster than MD5 on modern processors. Its inclusion will also improve interoperability between the Zcash blockchain and the EVM, and assist in future integration with IPFS, as they are working on switching their default hash function to BLAKE2b.

One BLAKE2 digest in Solidity, (see https://github.com/tjade273/eth-blake2/blob/optimise_everything/contracts/blake2.sol) currently requires `~480,000 + ~90*INSIZE` gas, and a single verification of an [Equihash](https://www.internetsociety.org/sites/default/files/blogs-media/equihash-asymmetric-proof-of-work-based-generalized-birthday-problem.pdf) PoW verification requires 2<sup>5</sup> to 2<sup>7</sup> iterations of the hash function, making use cases such as verification of Zcash block headers prohibitively expensive. However, an efficient implementation of BLAKE2b could allow verification of the Equihash PoW used in Zcash, making a BTC Relay - style SPV client possible on Ethereum.

Adding a precompile for the BLAKE2b hashing function will make the operation significantly faster and cheaper, improving usability.

## Parameters

* `METROPOLIS_FORK_BLKNUM`: TBD
* `GBLAKEBASE`: 30
* `GBLAKEWORD`: 6

## Specification

Adds a precompiled contract for the blake2b hash function.

Address of `0x9`

Function accepts a variable length input interpreted as:

    [OUTSIZE, D_1, D_2, ..., D_INSIZE]

where `INSIZE` is the length in bytes of the input. Throws if `OUTSIZE` is greater than 64. Returns the `OUTSIZE`-byte BLAKE2b digest, as defined in [RFC 7693](https://tools.ietf.org/html/rfc7693).

### Gas Costs

Gas costs would be equal to `GBLAKEBASE + GBLAKEWORD * floor(INSIZE / 32)`

## Rationale

The most frequent concern with EIPs of this type is that the addition of specific functions at the protocol level is a deviation from Ethereum's "featureless" design. It is true that a more elegant solution to the issue is to simply improve the scalability characteristics of the network so as to make calculating functions requiring millions of gas practical for everyday use. In the meantime, however, I believe that certain operations are worth subsidizing via precompiled contracts. There is significant precedent for this change, most notably the inclusion of the SHA256 precompiled contract, which is included largely to allow inter-operation with the Bitcoin blockchain.

Additionally, BLAKE2b is an excellent candidate for precompilation because of the extremely asymmetric efficiency which it exhibits. BLAKE2b is heavily optimized for modern 64-bit CPUs, specifically utilizing 24 and 63-bit rotations to allow parallelism through SIMD instructions and is little-endian. These characteristics provide exceptional speed on native CPUs: 3.08 cycles per byte, or 1 gibibyte per second on an Intel i5.

In contrast, the big-endian 32 byte semantics of the EVM are not conducive to efficient implementation of BLAKE2, and thus the gas cost associated with computing the hash on the EVM is disproportionate to the true cost of computing the function natively.

Note that the function can produce a variable-length digest, up to 64 bytes, which is a feature currently missing from the hash functions included in the EVM.

## Backwards Compatibility

There is very little risk of breaking backwards compatibility with this EIP, the sole issue being if someone were to build a contract relying on the address at `0x000....0000c` being empty. The likelihood of this is low, and should specific instances arise, the address could be chosen to be any arbitrary value, with negligible risk of collision.

In order to maintain backwards compatibility, the precompile will return `0` if `CURRENT_BLOCKNUM < METROPOLIS_FORK_BLKNUM`

The community response to this EIP has been largely positive, and besides the "no features" issue, there have as yet been no major objections to its implementation.

## Test Cases


## Implementation

The Go implementation is available from [golang.org](golang.org/x/crypto/blake2b) as well as [on github](https://github.com/dchest/blake2b).

Other languages:
[Javascript implementation](https://github.com/dcposch/blakejs)
[Java implementation](https://github.com/alphazero/Blake2b)
[Python implementatoin](https://github.com/buggywhip/blake2_py)

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
