### NewsData Event

News events offer invaluable insights into individual securities and the broader economy, often originating from sources external to the financial markets. This information is typically presented in text, elementized, or machine-readable formats and is sourced from reputable providers such as Dow Jones and Thomson Reuters.

BlitzTrader offers a robust framework to parse, normalize, and disseminate news information to your trading strategy. Whether it's an anticipated event triggering market reactions or unexpected developments impacting trading opportunities, BlitzTrader equips strategy developers with the tools to seamlessly integrate news events into their strategies. By subscribing to news providers, you can receive real-time News Data events as callbacks, empowering you to make informed trading decisions based on the latest market insights.

When a news event occurs, the OnNewsDataEvent method is triggered, providing access to crucial details such as the news ID, actual value and forecast value. 

```
string errorString = string.Empty;
if (SubscribeNewsDataUpdate("DOWJONES", out errorString))
{
    // Subscription to DowJones news provider successful
}
// More code...
protected override void OnNewsDataEvent(NewsDataEventArgs eventArgs)
{
    if (eventArgs.IsMachineReadableNews)
    {
        string newsID = eventArgs.ID;
        double actualValue = eventArgs.ActualValue;
        double forecastValueValue = eventArgs.ForecastValue;
    }
}
```

By incorporating real-time news data into your BlitzTrader strategies, you can gain a valuable edge in the ever-evolving market environment.