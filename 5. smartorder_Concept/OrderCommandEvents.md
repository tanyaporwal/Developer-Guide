### OrderCommand Events

In this crucial section, we explore the main events supported by the OrderCommand. The OrderCommand is pivotal in managing the order lifecycle, and any significant state change must trigger an alert via callback functions, providing finer control over the execution logic. Developers receive real-time alerts on different changes in the order state through these callback functions.

Below is a code snippet that defines and implements some order events:

```
protected override void OnInitialize()
{
            _ivInfoInstrument = base.GetIVInfo(_ivInstrument);

            _entryOrderCommand = base.GetSmartOrderCommand("Entry", TimeInForce.GFD, _ivInstrument);
            _exitOrderCommand = base.GetSmartOrderCommand("Exit", TimeInForce.GFD, _ivInstrument);

            _entryOrderCommand.OnOrderRejected += OnSmartOrderRejected;
            _entryOrderCommand.OnOrderSendingFailed += OnSmartOrderSendingFailed;
            _entryOrderCommand.OnOrderTraded += OnSmartOrderTraded;

            _exitOrderCommand.OnOrderRejected += OnSmartOrderRejected;
            _exitOrderCommand.OnOrderSendingFailed += OnSmartOrderSendingFailed;
            _exitOrderCommand.OnOrderTraded += OnSmartOrderTraded;
}


private void OnSmartOrderRejected(SmartOrderRejectedEventArgs orderRejected)
{
}

private void OnSmartOrderSendingFailed(SmartOrderSendngFailedEventArgs eventArgs)
 {
            if (eventArgs.OrderCommand.SmartOrderID.Equals(_entryOrderCommand.SmartOrderID))
            {
                // Stop the Strategy
                base.Stop("Entry Order Rejected. Reason : " +  eventArgs.Reason);
            }
            else
            {
               // Stop the Strategy
                base.Stop("Exit Order Rejected.. Reason : " +  eventArgs.Reason);
            }
        }
}

private void OnSmartOrderTraded(SmartOrderTradedEventArgs eventArgs)
{
}

> Key Information
1. When the Set method is initially called with True, it triggers the framework to initiate a Fresh New Order. Subsequent calls to the Set method before the order is acknowledged by the exchange will cache the new price in OrderCommand. Once the last order state is acknowledged as valid by the exchange, the framework will automatically initiate order modification based on the cached price.
2. Further calls to the Set method will modify the order with the new price only if the order is still in a clean open order state.
3. If the order reaches a terminal state with the order status being Completely Filled, Rejected, or Cancelled, any further Set calls will be ignored. Next order cycle can be initiated only after a Reset method call.
4. Once the SmartOrder reaches a terminal state, as mentioned in Step 3, you can reuse the SmartOrder for the next fresh order cycle only after calling the Reset method on IOrderCommand. Caution should be exercised while using the Reset method, as it can initiate a new order cycle.
5. Each IOrderCommand instance only opens one order cycle at a time. If your strategy requires multiple order cycles on the same instrument, define and initialize multiple IOrderCommand variables mapped to IVObject and use them accordingly.
6. f there's certainty that no order should be sent from the corresponding IOrderCommand at any point, it is safe not to Reset it.

This comprehensive guide ensures efficient order management and execution, enhancing trading strategies on BlitzTrader's platform.