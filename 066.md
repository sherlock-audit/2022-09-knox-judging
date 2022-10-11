Trumpero

high

# Wrong implementation of orderbook can make user can't get their fund back

## Lines of code 
https://github.com/sherlock-audit/2022-09-knox/blob/main/knox-contracts/contracts/auction/OrderBook.sol#L240
https://github.com/sherlock-audit/2022-09-knox/blob/main/knox-contracts/contracts/auction/OrderBook.sol#L182-L190

## Summary
When a user remove an order, next user call `addLimitOrder` can override the latest order with his/her order. It will make one who is owner of that latest order lose their fund. 

## Vulnerability Detail
Function `_remove` will decrease value of `index.length` by 1 when an order is removed
```solidity=
// url = https://github.com/sherlock-audit/2022-09-knox/blob/main/knox-contracts/contracts/auction/OrderBook.sol#L240
function _remove(Index storage index, uint256 id) internal returns (bool) {
    index.length = index.length > 0 ? index.length - 1 : 1;
    ...
}
```
Instead of reserving `id` of removed order to reuse for next created order, function `_insert` use the id of new order is `index.length + 1`
```solidity=
// url = https://github.com/sherlock-audit/2022-09-knox/blob/main/knox-contracts/contracts/auction/OrderBook.sol#L165-L172
function _insert(
    Index storage index,
    int128 price64x64,
    uint256 size,
    address buyer
) internal returns (uint256) {
    index.length = index.length > 0 ? index.length + 1 : 1;
    uint256 id = index.length;
    ...
}
```
It will override the latest order with new order's data.

For example
* Alice create an order with price = 10 --> `id = 1, index.length = 1`
* Bob create an order with price = 20 --> `id = 2, index.length = 2` 
* Alice cancel order `id = 1` --> `index.length = 1`
* Candice create new order with price = 30 
    * At this time, new order will have `id = index.length + 1 = 1 + 1 = 2`. It will override the state of Bob's order: price from 20 -> 30 

## Impact
User whose order is overrided can't withdraw their refund `ERC20` and their exercised tokens. 

## Code Snippet
```typescript=
it.only("bug", async() => {
    const totalContracts = await auction.getTotalContracts(epoch);

    const buyer1OrderSize = totalContracts.div(5);
    const buyer2OrderSize = totalContracts.div(5);
    const buyer3OrderSize = totalContracts.div(5);

    // buyer1 create order with price = 10
    await asset
      .connect(signers.buyer1)
      .approve(addresses.auction, ethers.constants.MaxUint256);
    await auction.addLimitOrder(epoch, 10, buyer1OrderSize);

    // buyer2 create order with price = 20
    await asset 
      .connect(signers.buyer2)
      .approve(addresses.auction, ethers.constants.MaxUint256);
    await auction
      .connect(signers.buyer2)
      .addLimitOrder(epoch, 20, buyer2OrderSize);

    // order with id = 2 have price = 20 
    expect( (await auction.getOrderById(epoch, 2)).price64x64 ).to.equal(20);

    // buyer1 cancel order with id = 1
    await auction.connect(signers.buyer1).cancelLimitOrder(epoch, 1);

    // buyer3 create order with price = 30
    await asset
      .connect(signers.buyer3)
      .approve(addresses.auction, ethers.constants.MaxUint256);
    await auction
      .connect(signers.buyer3)
      .addLimitOrder(epoch, 30, buyer3OrderSize);

    // order with id = 2 have price = 30 --> nervous 
    expect( (await auction.getOrderById(epoch, 2)).price64x64 ).to.equal(30);
});
```
To check with test, u can use this file 
https://gist.github.com/Trumpero/adbcd84c33f71856dbf379f581e8abbb
I write one more describe `::Bug` beside your original describe `::Auction` in file `Auction.behavior.ts` (just too lazy to write a new one). 

## Tool used
Hardhat 

## Recommendation
Use an array to store unused (removed) id, then assign each id to the new limit order created instead of using `index.length`. 
