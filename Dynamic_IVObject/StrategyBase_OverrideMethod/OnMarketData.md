### OnMarketDataEvent Override Method
The Strategy always requires access to real-time market data and alerts on changes in data state to evaluate opportunities triggering the order flow to the exchange. The Blitz Strategy development framework facilitates streaming real-time market data information for Level-I and Level-II data.

To receive real-time market data events for subscribed Instruments mapped with Strategy-defined IVObject, the Strategy developer needs to implement the OnMarketDataEvent method. The framework invokes this method in real-time whenever there is a price update of Level-I or Level-II market data of Instruments bound to subscribed IVObject.

```
OnMarketDataEvent method is one the important method that all Strategy must implements and puts their Strategy invocation logic.
```

Here's a sample implementation of the OnMarketDataEvent method:

```
protected override void OnMarketDataEvent(StrategyMarketDataEventArgs eventArgs)
{
            if (this.ActivateProfiling)
            {
                TraceLogInfo(string.Format("OnMarketDataEvent. Instrument: {0}, Time: {1}, LTP: {2}, LTQ: {3}",
                   eventArgs.IVInstrument.Instrument.DisplayName,
                   eventArgs.MarketDataContainerInfo.LastTradedTimeDT.ToString("dd-MM-yyyy HH:mm:ss"),
                   eventArgs.MarketDataContainerInfo.LastPrice, eventArgs.MarketDataContainerInfo.LastSize));
            }


            if (!_isStrikesLoaded && _ivInfo.MarketDataContainer.LastPrice > 0)
            {
                TraceLogInfo(string.Format("Subscribing market data for 25 up and down strikes"));
                if (LoadOptions())
                    _isStrikesLoaded = true;
            }

            DateTime ltd = _ivInfo.MarketDataContainer.TouchLineInfo.LastTradedTimeDT;

            double ltp = eventArgs.MarketDataContainerInfo.TouchLineInfo.LastPrice;
            double ltq = eventArgs.MarketDataContainerInfo.TouchLineInfo.LastPrice;
            double priceAtLevel5 = _ivInfo.MarketDataContainer.GetAskPriceAt(4);
            double tickSize = _ivInfo.IVInstrument.Instrument.TickSize;
            int netPosition = _ivInfo.Statistics.NetPosition;
            if(_ivInfo.RealizedMTM < Input_MaxMTMLoss)
            {
                ….
                ….
            }

            ……..
            ……..

}
```

In this implementation:
-	The OnMarketDataEvent method is overridden to handle incoming market data events.
-	Inside the method, relevant information such as instrument name, bid price, ask price, and volume is extracted from the eventArgs parameter.
-	The extracted data is then used to perform strategy invocation logic based on the received market data. This could involve evaluating trading opportunities, adjusting strategy state, or any other relevant actions based on the market conditions.