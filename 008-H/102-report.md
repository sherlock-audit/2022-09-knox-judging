0xNazgul

medium

# [NAZ-M6] Unbounded loop in `_previewWithdraw() && _redeemMax()` Can Lead To DoS

## Summary
There are some unbounded loops that can lead to DoS.

## Vulnerability Detail
The loop inside of `_previewWithdraw()` goes through all orders in `orderbook` and checks if the `data.buyer` is the `buyer` passed in parameter. It does some checks, math and an external call to remove the `data.id` from the orderbook. With all of this happening in the loop and costing gas it may revert due to exceeding the block size gas limit. This is the same case for `_redeemMax()` but has more gas costly executions with `transfer()` being involved.

## Impact
There are over thousands of orders that the loop has to go through, along with the massive amount of orders that are the `buyer`s. Half way through the execution fails due to exceeding the block size gas limit.

## Code Snippet
[`AuctionInternal.sol#L298`](https://github.com/sherlock-audit/2022-09-knox/blob/main/knox-contracts/contracts/auction/AuctionInternal.sol#L298), [`QueueInternal.sol#L140`](https://github.com/sherlock-audit/2022-09-knox/blob/main/knox-contracts/contracts/queue/QueueInternal.sol#L140)

## Tool used
Manual Review

## Recommendation
Consider avoiding all the actions executed in a single transaction, especially when calls are executed as part of a loop.