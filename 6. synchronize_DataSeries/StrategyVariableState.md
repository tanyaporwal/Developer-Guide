### Persist Strategy Variable State

In certain scenarios, it becomes necessary for strategy developers to save the state of strategy variables, allowing for recovery across multiple runs of the strategy instance. While Blitz typically handles the storage of important strategy states, custom state-saving mechanisms may need to be implemented.
The Blitz API offers two methods for saving and retrieving variable states: SaveStrategySetting and GetStrategySettings.

```
private const string TotalTradeCounterKey = "TotalTradeCounterKey";
private int _totalTradeCounter = 0;

protected override void OnInitialize()
{
    object totalTradeCounterObject = base.GetStrategySettings(TotalTradeCounterKey);
    if (totalTradeCounterObject != null)
        _totalTradeCounter = (int)totalTradeCounterObject;
}

protected override void OnTrade(IVObject ivObject, TradeDataEventArgs eventArgs)
{
    _totalTradeCounter++;
    base.SaveStrategySettings(TotalTradeCounterKey, _totalTradeCounter);
}
```

In this example, TotalTradeCounterKey is a unique identifier name for storing and retrieving the state of the _totalTradeCounter variable. During initialization, the saved value of _totalTradeCounter is retrieved using GetStrategySettings and assigned to the variable. During trading events, such as OnTrade, the _totalTradeCounter is updated and then saved using SaveStrategySettings to persist its state for future runs.
