cccz

medium

# A malicious keeper can manipulate the LToken's pricePerShare to take an unfair share of future users' deposits

## Summary
A well known attack vector for almost all shares based liquidity pool contracts, where an early user(keeper) can manipulate the price per share and profit from late users' deposits because of the precision loss caused by the rather large value of price per share.
## Vulnerability Detail
Only the keeper can call the initializeEpoch function of the VaultAdmin contract to deposit asset tokens into the Vault.
In the first epoch, a malicious keeper can 'deposit' with 1 wei of asset token as the first depositor of the Vault, and get 1 wei of shares.
Then the keeper can send 10000e18 - 1 of asset tokens and inflate the price per share from 1.0000 to an extreme value of 1.0000e22 ( from (1 + 10000e18 - 1) / 1) .
In later epochs, the shares gotten in processDeposits may be 0 due to loss of precision.
## Impact
The attacker can profit from future users' deposits. While the late users will lose part of their funds to the attacker.

## Code Snippet
https://github.com/sherlock-audit/2022-09-knox/blob/main/knox-contracts/contracts/vault/VaultAdmin.sol#L234-L243
https://github.com/sherlock-audit/2022-09-knox/blob/main/knox-contracts/contracts/queue/Queue.sol#L202-L217
https://github.com/solidstate-network/solidstate-solidity/blob/master/contracts/token/ERC4626/base/ERC4626BaseInternal.sol#L219-L232
https://github.com/solidstate-network/solidstate-solidity/blob/master/contracts/token/ERC4626/base/ERC4626BaseInternal.sol#L142-L149
https://github.com/solidstate-network/solidstate-solidity/blob/master/contracts/token/ERC4626/base/ERC4626BaseInternal.sol#L41-L59
https://github.com/sherlock-audit/2022-09-knox/blob/main/knox-contracts/contracts/vault/VaultInternal.sol#L156-L163
https://github.com/sherlock-audit/2022-09-knox/blob/main/knox-contracts/contracts/vault/VaultInternal.sol#L102-L125
## Tool used

Manual Review

## Recommendation

Uniswap V2 solved this problem by sending the first 1000 LP tokens to the zero address https://github.com/Uniswap/v2-core/blob/master/contracts/UniswapV2Pair.sol#L119-L124