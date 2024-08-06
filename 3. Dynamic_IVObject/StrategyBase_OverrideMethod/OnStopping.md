### OnStopping Override Method
The method is called when the user explicitly stops the Strategy instance from the trading dashboard. Additionally, this method is automatically invoked when the Blitz system detects the rejection of an order in a looping state, preventing the continuous pumping of orders towards the exchange.
Within this method, Strategists can perform cleaning tasks, such as canceling all open orders or squaring off any open positions. 

```
protected override void OnStopping()
 {
    foreach (IOrderCommandEx orderCommandEx in _orderExecutionMAP.Values)
    {
        if (orderCommandEx.IVInfo.Statistics.NetPosition != 0)
        {
            orderCommandEx.UserText = "Strategy Stopped.";
            orderCommandEx.GoFlat();
        }
      }

      TraceLogInfo("Strategy Stopped.");
}
```

In the above code snippet, an advanced execution concept of Blitz is utilized, which allows for the squaring off of all open positions across any market with a simple call. We will delve into the order state management and execution functionality of Blitz in later sections.

Furthermore, the Strategy developer can also explicitly stop the strategy with an API call from their Strategy code.

```
if (isStrategyStoppedTimeBreached)
    base.Stop("The startegy is stopped due to EOD time reached.");
```

The following code snippet checks the Strategy mode and may return before evaluating core trading logic if the Strategy is not in the started state:

```
if(!this.StrategyRunningMode == StrategyMode.Started)
{
    return;
}
```