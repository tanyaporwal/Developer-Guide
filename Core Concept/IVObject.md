## The IVObject, the core of the strategy
Every trading strategy requires a market or instrument on which it listens for real-time market data information and decides to place orders as opportunities arise. In the BlitzTrader Strategy development framework, a fundamental concept known as IVObject (Instrument Variable Object) is introduced, which serves as the centerpiece of your strategy and abstractly defines the market. This concept enables building strategies in a market-neutral manner, allowing the framing of strategy logic without hard-coding the specific market on which the strategy ultimately executes.
IVObject facilitates creating strategies that can be applied across various markets or instruments without modification. It supports the development of both simple execution strategies, where opportunities are based on predefined market conditions, as well as more complex strategies such as options trading, where multiple strike price instruments are involved.
IVObject serves as a market-neutral representation of the instrument across any market or exchange, allowing strategies to remain adaptable and scalable across different trading environments.
The strategy framework offers support for both statically and dynamically defining IVObject variables. Static IVObject declaration and initialization are typically employed in simpler strategies, while dynamic IVObject creation becomes essential in more complex scenarios, such as options trading strategies involving multiple strike price instruments.

> IVObject is market neutral representation of the instrument across any market or exchange.

Here's an example of how to define a static IVObject variable named _ivInstrumentFutures in your strategy class.

private IVObject _ivInstrumentFutures = new IVObject(
                                   "FuturesIV",                              // The Identifier Name
                                   "Futures Instrument",            // The description
                                    true,                                         // True means IV is tradable
                                    InstrumentType.Futures,      // The Instrument type Futures means only Futures instrument can be mapped to IVObject
                                    MarketDataType.All,            // The strategy subscribed for all the market data events
                                    OrderEventType.All);            // The strategy subscribe for all the order event call back

The first argument represents the unique identifier name. This should be unique name across different instrument variable within the same strategy implementation class.

The fourth argument represents the asset class type of instrument variable:

| Member Name | Value | Description |
| :--- | :--- | :--- |
| Futures | 1 | Represent Futures Contract |
| Options | 2 | Represent Options Contract | 
| Spread | 4 | Represent 2-Leg Spread Contract | 
| Equity | 8 | Represent Equity Contract | 
|Spot|16|Represent Spot Contract (Forex)|

The fifth argument requiredMarketDataType used to declare for the kind of market data event for the IVObject. The strategy must declare a callback function OnMarketDataEvent to receive registered market data events.

override void OnMarketDataEvent(StrategyMarketDataEventArgs strategyMarketDataEventArgs)

Following are the events type that can be registered:

| Member Name | Value | Description |
| :--- | :--- | :--- |
| None | 1 | Represent no market data event delivered as a callback event |
| Touchline | 2 | Represent level-I market data event delivered as a callback event |
| MarketDepth | 4 | Represent level-II market data event delivered as a callback.event |

The fifth argument requiredOrderEventType  allow programmer to register for different order events that triggers during the order state changes during the order life cycle. This allows programmers to receive the order events as a callback function for a respective order state change.

| Member Name | Value	| Description |
| :--- | :--- | :--- |
| None | 1 | Represent no order event delivered as a callback |
| OrderAccepted | 2 | Represent an order accepted event by exchange is delivered as a callback method:
protected override void OnOrderAccepted(IVObject ivObject, OrderAcceptedEventArgsorderAcceptedEventArgs) |
| OrderRejected | 4 | Represent an order rejected event by exchange is delivered as a callback method:
protected override void OnOrderRejected(IVObject ivObject, OrderRejectedEventArgsorderRejectedEventArgs) |
| OrderModificationAccepted | 8 | Represent an order modification request accepted by exchange is delivered as a callbackmethod:
protected override void OnOrderModified(IVObject ivObject,OrderModificationAcceptedEventArgs orderModificationAcceptedEventArgs) |
| OrderModificationRejected | 16 | Represent a order modification request rejected event by exchange is delivered as acallback method:
protected override void OnOrderModificationRejected(IVObject ivObject,OrderModificationRejectEventArgs orderModificationRejectEventArgs) |
| OrderCancellationAccepted | 32 | Represent a order cancellation request accepted event by exchange is delivered as acallback method:
protected override void OnOrderCancelled(IVObject ivObject,OrderCancellationAcceptedEventArgs orderCancellationAcceptedEventArgs) |
| OrderCancellationRejected | 64 | Represent a order cancel request rejected event by exchange is delivered as a callbackmethod:
protected override void OnOrderCancellationRejected(IVObject ivObject,OrderCancellationRejectEventArgs orderCancellationRejectEventArgs) |
| OrderTraded | 128 | Represent a order traded event by exchange is delivered as a callback method:
protected override void OnTrade(IVObject ivObject, TradeDataEventArgs tradeDataEventArgs) |
| OrderTriggered | 256 | Represent a stop order triggered event by exchange is delivered as a callback method:
protected override void OnOrderTriggered(IVObject ivObject, OrderTriggeredEventArgseventArgs) |
| All | 510	| Represent all above event by exchange is delivered on their respective callback method |

If you are developing a pair trading strategy, you might require the declaration of two static IVObject variables, named _ivPair1 and _ivPair2.
Similarly, if you are developing a conversion reversal strategy, you might need to declare three static IVObject variables: _ivFuturesLeg, _ivCallOptionsLeg, and _ivPutOptionsLeg.
However, it's crucial to note that all static IVObject variables declared within the strategy must be returned as a collection from the following override method. This step is mandatory for proper recognition of the declared IVObjects by the Blitz trading platform.

```
public class MainStrategy : StrategyBase
{

    private IVObject _ivFuturesLeg = new IVObject("FuturesIV", "Futures Instrument", true,
                                                           InstrumentType.Futures,
                                                           MarketDataType.All, OrderEventType.All);

    private IVObject _ivCallLeg = new IVObject("CallIV", "Call Instrument", true,
                                                           InstrumentType.Options,
                                                           MarketDataType.All, OrderEventType.All);

    private IVObject _ivPutLeg = new IVObject("PutIV", "Put Instrument", true,
                                                           InstrumentType.Options,
                                                           MarketDataType.All, OrderEventType.All);


     

      public override IVObject[] IVObjects
      {
            get
            {
                List<IVObject> _ivObjectArray = new List<IVObject>();
                _ivObjectArray.Add(_ivFuturesLeg);
                _ivObjectArray.Add(_ivCallLeg);
                _ivObjectArray.Add(_ivPutLeg);

                return _ivObjectArray.ToArray();
            }
        }

}
```


> Note: All static IVObjects are mapped to actual tradable instruments by the trader from the trading dashboard using a process called Instance Creation of Strategy. This mapping process can also be automated using a Strategy Plugin concept programmed within the UI dashboard using plugin SDK.

> For instance, a simple automation could involve creating a strategy instance automatically from a user-defined CSV file that specifies the instrument to be mapped to the IVObject, along with initializing default input variable values. This automation streamlines the setup process and ensures that strategies can be deployed efficiently across different instruments or markets.
