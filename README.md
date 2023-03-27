# Trading Engine

Order Matching / Trading Engine
- Built with .Net Core, allowing run on linux and windows machines
- Supports multiple order types such as:
  - Limit
  - Market
  - Stop Loss / Stop Limit
  - Immediate or Cancel (IOC)
  - Fill or Kill (FOK)
  - Good Till Date (GTD)
  - Iceberg
  
**Hand written serializer faster than any serializer. x15 times faster than JSON, x5 times faster than messagepack**

## Order Types
### Order Fields
**IsBuy**

true if buy/bid, false if sale/ask

**OrderId**

Unique number assigned to each order. uniquely identifies each order.

**OpenQuantity**

Quantity user is willing to buy/sell. MatchingEngine will reduce OpenQuantity after each match.

**Price**

Limit price of the order, in case of buy maximum price buyer willing to pay for one unit/quantity of the asset. In case of sale minimum price seller is willing to accept for one unit/quantity of the asset. In the case of market order it must be 0.

**StopPrice**

trigger price for stop order.

**CancelOn**

Seconds from 1 Jan 1970(UTC) after which order should be cancelled. This should be set for only good till date order. for good till cancel orders it must be set to 0.

**OrderCondition**

Represents order condition, it could be one of the following:
1. Immediate or cancel - Order must be filled immediately, if order is not filled then cancel the remaining open order. it allows partial fills.
2. Fill or kill - only fill order in full immediately or cancel the order.
3. Book or cancel - Order must not be match with any open order while placing order. it must be added to order book. it can not be matched with resting orders in book. if order can be matched resting order in book then cancel the order.

**TipQuantity**

Tip quantity for each tip added into order book. This must be set for iceberg orders. for other orders it must be set to 0.

**TotalQuantity**

Total quantity of iceberg order. This must be set for iceberg orders. for other orders it must be set to 0.

**OrderAmount**

OrderAmount can be specified for market buy and stop market buy. instead of providing OpenQuantity we could provide amount. for example for market buy bitcoin worth of 2000 USD. it means buy any quantity of bitcoin at any price but amount should not exceed 2000 USD.

**FeeId**

represents fee to be charged for this order.

**Fee**

for order matcher internal user. order matcher calculates fee on order.

**Sequence**

for order matcher internal use. Sequence is used to find priority of order. Lower sequence has higher priority.

**Cost**

for order matcher internal use. order matcher tracks cost of the order. it is sum of matched quantity * match rate of all the matches.

## Order Examples

**Limit**
```C#
Order limitBuyOrder = new Order { IsBuy = true, OrderId = 1, OpenQuantity = 1000, Price = 10 };
Order limitSellOrder = new Order { IsBuy = false, OrderId = 2, OpenQuantity = 1000, Price = 10 };
```
Buy or sell 1000 shares at a price of $10.00.

**Stop Limit**
```C#
Order stopLimitBuy = new Order { IsBuy = true, OpenQuantity = 1000, Price = 11, OrderId = 1, StopPrice = 10 };
Order stopLimitSell = new Order { IsBuy = false, OpenQuantity = 1000, Price = 8, OrderId = 1, StopPrice = 9 };
```
Order to buy 1000 shares after the price goes above $10.00 and is willing to pay $11.00 a share. Or willing to sell 1000 shares after the price goes below $9.00 and is willing to accept $8.00 per share

**Market**
```C#
Order marketSell  = new Order { IsBuy = false, OpenQuantity = 500, Price = 0, OrderId = 2 };
Order marketBuy = new Order { IsBuy = true, Price = 0, OrderId = 5, OrderAmount = 5000 };
```
Sell 500 shares at market price once order is routed, or buy any quantity of shares at any price but for an order amount total of $5000.00.

**Stop Market / Stop Loss**
```C#
Order stopMarketSell = new Order { IsBuy = false, OpenQuantity = 500, Price = 0, OrderId = 3, StopPrice = 9 };
Order stopMarketBuy = new Order { IsBuy = true, Price = 0, OrderId = 3, StopPrice = 11, OrderAmount = 5500 };
```
Order will be triggered when the price goes below $9.00 and iwll sell 500 shares the current bid price. Order will be triggered when the price goes above $11.00 and will buy shares of a total of $5500

**Iceberg Order**
```C#
Order order = new Order { IsBuy = false, OpenQuantity = 500, Price = 10, OrderId = 1, TipQuantity = 500, TotalQuantity = 5000 };
```
The above order is willing to sell 5000 shares at a price of $10.00 each, but the order will show a tip of 500 shares only. Therefore only 500 shares will be visible on order book.

**Good till Date**
```C#
Order gtdOrder = new Order { IsBuy = true, OpenQuantity = 500, Price = 10, OrderId = 1, CancelOn = 1 };
```
The order will automatically be cancelled on January 1st, 1970 UTC. Orders can be cancelled manually aswell before the CancelOn time.

**Immediate or Cancel**
```C#
Order iocOrder = new Order { IsBuy = true, OpenQuantity = 1500, Price = 10, OrderId = 2, OrderCondition = OrderCondition.ImmediateOrCancel };
```
MatchingEngine will match what ever possible and cancel remaining quantity.
