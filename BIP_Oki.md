<pre>
  BIP: Oki
  Layer: Consensus (hard fork)
  Title: Dynamic maximum block weight by miner vote
  Author: Oki Burokku 
  Comments-Summary: No comments yet.
  Comments-URI: https://github.com/oki-burokku/The-White-Paper
  Status: Requires Oki Consensus
  Type: Standards Track
  Created: 2020-06-01
  License: BSD-2-Clause
</pre>

==Abstract==

Replace the static 4M block weight hard limit with a hard limit set by coinbase vote, conducted on the same schedule as difficulty retargeting.

==Motivation==

Miners directly feel the effects, both positive and negative, of any maximum block weight change imposed by their peers.  Larger blocks allow more growth in the on-chain ecosystem, while smaller blocks reduce resource requirements network-wide.  Miners also act as an efficient proxy for the rest of the ecosystem, since they are paid in the tokens collected for the blocks they create.

A simple deterministic system is specified, whereby a 75% mining supermajority may activate a change to the maximum block weight each 2016 blocks.  Each change is limited to a 4M increase from the previous block weight hard limit, or a decrease of similar magnitude.  Among adopting nodes, there will be no disagreement on the evolution of the maximum block weight.

==Specification==

===Dynamic Maximum Block Size===
# Initial value of <code>hardLimit</code> is 1000000 bytes, preserving current system.
# Changing <code>hardLimit</code> is accomplished by encoding a proposed value, a vote, within a block's coinbase scriptSig, and by processing the votes contained in the previous retargeting period.<br /><br />
## Vote encoding
### A vote is represented as a megabyte value using the BIP100 pattern<br /><br /><code>/BIP100/B[0-9]+/</code><br /><br />Example: <code>/BIP100/B8/</code> is a vote for a 8000000-byte <code>hardLimit</code>.<br /><br />
### If the block height is encoded at the start of the coinbase scriptSig, as per BIP34, it is ignored.
### Only the first BIP100 pattern match is processed in "Maximum block size recalculation" below.
### A megabyte value is represented by consecutive base-ten digits.
### If no BIP100 pattern is matched, the first matching emergent consensus pattern <code>/EB[0-9]+/</code>, if any, is accepted as the megabyte vote.<br /><br />
## Maximum block size recalculation
### A <code>new hardLimit</code> is calculated after each difficulty adjustment period of 2016 blocks, and applies to the next 2016 blocks.
### Absent/zero-valued votes are counted as votes for the <code>current hardLimit</code>.
### The votes of the previous 2016 blocks are sorted by megabyte vote.
### Raising <code>hardLimit</code><br /><br />
#### The <code>raise value</code> is defined as the vote of the 1512th highest block, converted to bytes.
#### If the resultant <code>raise value</code> is greater than (<code>current hardLimit</code> * 1.05) rounded down, it is set to that value.
#### If the resultant <code>raise value</code> is greater than <code>current hardLimit</code>, the <code>raise value</code> becomes the <code>new hardLimit</code> and the recalculation is complete.<br /><br />
### Lowering <code>hardLimit</code><br /><br />
#### The <code>lower value</code> is defined as the vote of the 1512th lowest block, converted to bytes.
#### If the resultant <code>lower value</code> is less than (<code>current hardLimit</code> / 1.05) rounded down, it is set to that value.
#### If the resultant <code>lower value</code> is less than <code>current hardLimit</code>, the <code>lower value</code> becomes the <code>new hardLimit</code> and the recalculation is complete.<br /><br />
### Otherwise, <code>new hardLimit</code> remains the same as <code>current hardLimit</code>.

===Signature Hashing Operations Limits===
# The per-block signature hashing operations limit is scaled to (actual block size, fractional megabyte rounded to next higher megabyte) / 50.
# A maximum serialized transaction size of 1000000 bytes is imposed.

==Recommendations==

===Publication of <code>hardLimit</code>===
# For the benefit of all observers, it is recommended that <code>hardLimit</code> be published.  Example: a complete coinbase string might read <br /><br /><code>/BIP100/B8/EB2.123456/</code><br /><br /> which indicates a vote for 8M maximum block size, and an enforced <code>hardLimit</code> of 2.123456 megabytes for the block containing the coinbase string.

==Deployment==

This BIP is presumed deployed and activated as of block height 449568 by implementing nodes on the bitcoin mainnet. It has no effect until a raise value different from 1M is observed, which requires at least 1512 of 2016 blocks to vote differently from 1M.

==Backward compatibility==

The first block larger than 1M will create a network partition, as nodes with a fixed 1M hard limit reject that block.

==Implementations==
https://github.com/oki-burokku/bitcoin/pull/bbb</br>

==Copyright==
This document was based on the BIP100 proposal by Jeff Garzik, Tom Harding & Dagur Valberg Johannsson.
This document is licensed under the BSD 2-clause license.
