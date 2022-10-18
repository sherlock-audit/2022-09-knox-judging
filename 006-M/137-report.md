berndartmueller

medium

# Chainlink's `latestRoundData` might return stale or incorrect results

## Summary

Chainlink's `latestRoundData()` is used but there is no check if the return value indicates stale data. This could lead to stale prices according to the Chainlink documentation:

- https://docs.chain.link/docs/historical-price-data/#historical-rounds

## Vulnerability Detail

The `PricerInternal._latestAnswer64x64` function uses Chainlink's `latestRoundData()` to get the latest price. However, there is no check if the return value indicates stale data.

## Impact

The `PricerInternal` could return stale price data for the underlying asset.

## Code Snippet

[PricerInternal.\_latestAnswer64x64](https://github.com/sherlock-audit/2022-09-knox/blob/main/knox-contracts/contracts/pricer/PricerInternal.sol#L50-L52)

```solidity
/**
  * @notice gets the latest price of the underlying denominated in the base
  * @return price of underlying asset as 64x64 fixed point number
  */
function _latestAnswer64x64() internal view returns (int128) {
    (, int256 basePrice, , , ) = BaseSpotOracle.latestRoundData();
    (, int256 underlyingPrice, , , ) =
        UnderlyingSpotOracle.latestRoundData();

    return ABDKMath64x64.divi(underlyingPrice, basePrice);
}
```

## Tool Used

Manual review

## Recommendation

Consider adding checks for stale data. e.g

```solidity
(uint80 roundId, int256 basePrice, , uint256 updatedAt, uint80 answeredInRound) = BaseSpotOracle.latestRoundData();

require(answeredInRound >= roundId, "Price stale");
require(block.timestamp - updatedAt < PRICE_ORACLE_STALE_THRESHOLD, "Price round incomplete");
```
