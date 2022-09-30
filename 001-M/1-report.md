8olidity
# Calls inside loops that may address DoS

## Summary
Calls inside loops that may address DoS

## Vulnerability Detail
Calls inside loops that may address DoS
## Impact
Calls to external contracts inside a loop are dangerous (especially if the loop index can be user-controlled) because it could lead to DoS if one of the calls reverts or execution runs out of gas.
[Reference](https://swcregistry.io/docs/SWC-113)

## Code Snippet
https://github.com/None/blob/None/knox-contracts/contracts/queue/QueueInternal.sol#L140-L150

```
    function _redeemMax(address receiver, address owner) internal {
        uint256[] memory tokenIds = _tokensByAccount(owner);
        uint256 currentTokenId = QueueStorage._getCurrentTokenId();

        for (uint256 i; i < tokenIds.length; i++) {
            uint256 tokenId = tokenIds[i];
            if (tokenId != currentTokenId) {
                _redeem(tokenId, receiver, owner);
            }
        }
    }
```
## Tool used
vscode
Manual Review

## Recommendation
Avoid combining multiple calls in a single transaction, especially when calls are executed as part of a loop Always assume that external calls can fail Implement the contract logic to handle failed calls