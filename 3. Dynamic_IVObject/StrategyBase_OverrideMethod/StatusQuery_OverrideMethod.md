### OnStrategyStatusQuery Override Method

Sometimes, the Quant requires informative updates on the state of a Strategy instance at any given time, triggered from the Trading Dashboard. The Blitz Trading Dashboard features a predefined command called Status Query, accessible via the context menu for each Strategy instance. Upon invocation from the Trading Dashboard, control is passed to the callback function OnStrategyStatusQuery within your Strategy Instance.

Developers have the flexibility to reflect the current state of the Strategy Instance using the TraceLogInfo function. Any text logged using TradeLogInfo will be displayed in the Log Window within the Trading Dashboard. This process facilitates the creation of informative states to be reflected in the Trading Dashboard as needed.

```
protected override void OnStrategyStatusQuery()
{
            TraceLogInfo ("OnStrategyStatusQuery.”);

            double straddleExecutionPrice = 0;
            double straddleMarketPrice = 0;

            foreach (IOrderCommandEx orderCommandEx in _orderExecutionMAP.Values)
            {
                if(orderCommandEx.IVInfo.Statistics.NetPosition != 0)
                {
                    straddleExecutionPrice += orderCommandEx.GetAverageExecutionPrice();
                    straddleMarketPrice += orderCommandEx.IVInfo.MarketDataContainer.LastPrice;


                    TraceLogInfo(string.Format("Options status. CallInstrument: {0}, Position: {1}, ExecutionPrice: {2}, LTP: {3}, InstrunctionType: {4}",
                        _callOptionsOrderCommandEx.IVInfo.InstrumentName,
                        _callOptionsOrderCommandEx.IVInfo.Statistics.NetPosition,
                        _callOptionsOrderCommandEx.GetAverageExecutionPrice(),
                        _callOptionsOrderCommandEx.IVInfo.MarketDataContainer.LastPrice,
                        _callOptionsOrderCommandEx.InstructionType.ToString()));
                }
            }

            TraceLogInfo (string.Format("StraddleExecutionPrice: {0}, StraddleMarketPrice: {1}, MTM: {2}",
                straddleExecutionPrice,
                straddleMarketPrice,
                base.UnrealizedMTM + base.RealizedMTM));
}
```
> The provided code snippet collects crucial state information of the Strategy and relays it to the Trading Dashboard through TraceLogInfo.

