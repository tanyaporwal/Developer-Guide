### MarketDataDelayNotifyTimeInSeconds Override Method

The Strategy developer sometimes needs an alert if real-time Market Data events are not reaching the Strategy for any reason.
They can utilize the MarketDataDelayNotifyTimeInSeconds method to specify the duration after which the alert mechanism will be invoked if market data is not received. In following example the alert mechanism is raised if market data event is not received for 10 seconds.

```
protected override int MarketDataDelayNotifyTimeInSeconds
{
    get { return 10; }
}
```

Additionally, they can override the OnMarketDataDelay method to handle the alert mechanism:
override  void OnMarketDataDelay(StrategyMarketDataDelayEventArgs eventArgs)
{
}

The OnMarketDataDelay callback provides the Strategy developer with an opportunity to manage system risk or raise alert mechanism  if the Strategy is stalled due to failures in the market data channel from the exchange or data provider.
