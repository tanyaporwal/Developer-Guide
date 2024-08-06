### Logging Mechanism in Strategy

The Strategy API's logging mechanism empowers your application to enable debugging information essential for diagnosing and troubleshooting issues effectively. Logging is a critical tool for debugging and diagnosing strategy-related issues, providing valuable insights into the application's behavior, current state, or important events.

BlitzTrader equips you with a robust logging API, enabling your strategies to record informative messages, warnings, and error messages.. These logs serve various purposes, from providing informative data about the strategy's current state to alerting traders about critical events or potential issues.

BlitzTrader's logging API includes the following methods:

| |
| :--- |
| TraceLogInfo(String infoText) |
| TraceLogWarning(String warningText) |
| TraceLogError(String errorText) |
Moreover, any text logged using the logging API will be displayed in the Trading Dashboard, offering real-time visibility into the strategy's activity. Additionally, the logged information is automatically stored in a Strategy file logger, ensuring comprehensive record-keeping and facilitating further analysis and review.

```
protected override void OnStopping()
{
    TraceLogInfo("Strategy Stopped.");
}
```

Developers can choose to enable conditional logging based on a pre-built input parameter called ActiveProfiling. This allows for strategic log placement, avoiding excessive logging that might impact performance. The value of ActiveProfiling can be set true or false from the Trading Dashboard Strategy Input parameter window.

```
protected override void OnMarketDataEvent(StrategyMarketDataEventArgs eventArgs)
{
    if (base.ActivateProfiling)
    {
        base.TraceLogInfo(string.Format("New Market Data -> Instrument:{0}, BID:{1:0.00}, ASK:{2:0.00}, LTP:{3:0.00}, LTPTime:{4}",
                                                                    eventArgs.MarketDataContainerInfo.Instrument.DisplayName,
                                                                    eventArgs.TouchLineInfo.BestBidPrice,
                                                                    eventArgs.TouchLineInfo.BestBidPrice,
                                                                    eventArgs.TouchLineInfo.LastPrice,
                                                                    eventArgs.TouchLineInfo.LastTradedTimeDT.ToString("dd-MMM-yyyy hh:mm:ss")));
    }
}
```