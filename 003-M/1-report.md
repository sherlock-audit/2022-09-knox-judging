Bnke0x0
# Low-level call return value not checked

## Summary
The tow functions '_wrapNativeToken()' in the (AuctionInternal.sol/QueueInternal.sol) they performs a low-level '.call' in 'payable(msg.sender).call{value: msg.value - amount}' but does not check the return value if the call succeeded.

## Vulnerability Detail

## Impact
If the call fails, the refunds did not succeed and the caller will lose all refunds of msg.value - amount.

## Code Snippet
1. File: https://github.com/None/blob/None/knox-contracts/contracts/auction/AuctionInternal.sol#L581

       'payable(msg.sender).call{value: msg.value - amount}("");'

2. File: https://github.com/None/blob/None/knox-contracts/contracts/queue/QueueInternal.sol#L193

       'payable(msg.sender).call{value: msg.value - amount}("");'

## Tool used

Manual Review

## Recommendation
Revert the entire transaction if the refund call fails by checking that the success return value of the payable(to).call(...) returns true.