### Order and Executions Reports Callback

Order flow management is a fundamental requirement for any Strategy, and Blitz OMS (Order Management System) is adept at managing position and order bookkeeping, alerting the Strategy to any changes in order status through a callback mechanism.

Below are the important callback functions that execute upon changes in order status:
These callback functions are crucial for the Strategy to react promptly to changes in order status, enabling efficient management of trades and positions within the trading system

```
protected override void OnOrderAccepted(IVObject ivObject, OrderAcceptedEventArgs eventArgs) { }

this function is triggered when an order is accepted by the liquidity provider and assigned a valid Exchange Order ID. Your strategy can then take appropriate actions, such as further action to modify or cancel the order.
```

```
protected override void OnOrderRejected(IVObject ivObject, OrderRejectedEventArgs eventArgs) { }

This function is invoked when an order is rejected. Possible reasons for rejection include:
-	Internal risk management check failure within BlitzTrader.
-	Loss of connectivity with the exchange by the exchange adapter.
-	Missing mandatory information in the order.
-	Rejection by the exchange itself due to margin constraints or non-compliance with exchange rules.
```

```
protected override void OnOrderModified(IVObject ivObject, OrderModificationAcceptedEventArgs eventArgs) { }

This function is triggered when a modification request for an open order is accepted by the exchange.
Common Order Modifications:
Price Changes: Your strategy can update the price of an open order to adapt to changing market conditions.
Quantity Adjustments: Modify the order quantity to better align with your trading goals.
```

```
protected override void OnOrderCancelled(IVObject ivObject, OrderCancellationAcceptedEventArgs eventArgs) { }

This function is invoked when your order cancellation request is acknowledged, and the order is gracefully cancelled. This represents a terminal order state, and Blitz will remove the order from the open order book. The one of the reason for Strategy to cancel the open order is that your strategy can proactively exit positions when market conditions turn unfavorable.
```

```
protected override void OnOrderCancellationRejected(IVObject ivObject, OrderCancellationRejectEventArgs eventArgs) { }

This method is invoked when your request for order cancellation is rejected. This event is rare and may occur due to the unmatched state of your order with the exchange. 

One likely possibility of this event is:
Order Already Filled: It's possible that your cancellation request arrives after the order has already been completely filled by the exchange. In this case, the order has reached a terminal state, and there's nothing to cancel. The exchange will reject the cancellation request since the order is no longer active.
```

```
protected override void OnTrade(IVObject ivObject, TradeDataEventArgs eventArgs) { }

This method is invoked when the exchange generates a trade event in the form of a partial or completely filled sequence. It provides notification to the Strategy instance about trades executed on the associated order.
```

```
protected override void OnOrderTriggered(IVObject ivObject, OrderTriggeredEventArgs eventArgs){ }
```