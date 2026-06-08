```
  BIP: ???
  Layer: Consensus (soft fork)
  Title: OP_SINGLETON
  Author: Joshua Doman <joshsdoman@gmail.com>
  Status: Draft
  Type: Specification
  License: BSD-3-Clause
  Requires: ??? (Pay-to-Singleton)
```

## Abstract

This document proposes a new operation for [Tapscript][tapscript-bip]: `OP_SINGLETON`. It introduces the ability to push onto the stack the identifier of a [Singleton][singleton-bip] output spent at a specific index.

## Motivation

`OP_SINGLETON` can be used to commit to the transaction spending a [Singleton][singleton-bip] output with a specific identifier. This capability can be used to create a signature-less [contract-level relative timelock][clrt], which [improves a variant][ln-symmetry-variant] of LN-Symmetry that uses Singleton outputs to eliminate the [2x-delay problem][2x-delay-problem].

In addition, `OP_SINGLETON` can be used alongside `OP_CSFS` [BIP348][csfs-bip] to delegate to a Singleton without specifying the current UTXO. This improves the architecture of rollup proposals like [ShieldedCSV][shielded-csv] by enabling users to non-interactively add funds [through delegation][bridging].

## Specification

`OP_SINGLETON` redefines `OP_SUCCESS207` (0xcf) in the Tapscript execution context with further restrictions.

Upon execution of the opcode, there must be at least one element on the stack, and the top element must be `<index>`, a minimally encoded integer. `OP_SINGLETON` pops the top element and pushes the 32-byte Singleton identifier spent at `<index>`, as defined in [Pay-to-Singleton][singleton-bip], onto the stack.

Execution fails if:
- `<index>` is negative or greater than or equal to the number of inputs.
- The output spent at `<index>` is not a version 3 SegWit output with a 32-byte program.
- The witness at `<index>` has fewer than 2 elements.
- The last element after removing the optional annex (as defined in [Pay-to-Singleton][singleton-bip]) has fewer than 33 bytes.

This last element after removing the optional annex is called the control block, and the pushed identifier is `control[1:33]`.

## Backward compatibility

This document proposes to give meaning to a Tapscript `OP_SUCCESS` operation. The presence of an `OP_SUCCESS` would previously make a Tapscript execution unconditionally succeed. This proposal therefore only tightens the block validation rules: there is no block that is valid under the rules proposed in this BIP but not under the existing Bitcoin consensus rules. As a consequence these changes are backward-compatible with non-upgraded node software.

## Implementation

* https://github.com/joshdoman/bitcoin/tree/202606-op-singleton

## Test Vectors

TODO

## Acknowledgements

This BIP draws language from [BIP349][bip-349] and [BIP446][bip-446].

## Copyright

This document is licensed under the 3-clause BSD license.


[singleton-bip]: bip-XXXX-pay-to-singleton.mediawiki
[tapscript-bip]: bip-0342.mediawiki
[clrt]: https://delvingbitcoin.org/t/contract-level-relative-timelocks-or-lets-talk-about-ancestry-proofs-and-singletons/1353
[ln-symmetry-variant]: bip-XXXX-pay-to-singleton.mediawiki#lightning-symmetry-without-2x-delay
[2x-delay-problem]: https://bitcoinops.org/en/newsletters/2025/01/10/#contract-level-relative-timelocks
[csfs-bip]: bip-0348.md
[shielded-csv]: https://eprint.iacr.org/2025/068
[bridging]: bip-XXXX-pay-to-singleton.mediawiki#non-interactive-bridging
[bip-349]: bip-0349.md
[bip-446]: bip-0446.md