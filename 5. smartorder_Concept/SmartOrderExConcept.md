### SmartOrderEx Concept
While OrderCommand (SmartOrder) provides granular control over order execution logic, IOrderCommandEx (SmartOrderEx)offers a higher level of abstraction, simplifying the management of order and position states within the Strategy by utilizing prebuilt  standard or proprietary  execution components.
SmartOrderEx presents a straightforward yet powerful approach to order execution using intuitive commands. For instance, developers can aim to achieve specific positions with simple API calls such as EnterLong, EnterShort, GoFlat, SetPosition, IncreaseLong, DecreaseLong, IncreaseShort, DecreaseShort, and more.

The framework seamlessly integrates user instructions to attain desired directions and positions, with the system handling the execution process. Additionally, SmartOrderEx provides flexibility by allowing developers to define an Execution Algorithm bound to IOrderCommandEx. This algorithm determines the slicing of order quantity and the quoting mechanism, tailored to specific trading strategies.

This mechanism empowers developers to create various execution algorithms tailored to their strategy's needs, leveraging market insights to define the aggressiveness of order execution.
The SmartOrderEx process requires developers to define two mandatory execution algorithms for order quantity slicing and price quoting. Below, we delve into the implementation of these two algorithms from scratch.

Below is the declaration of SmartOrderCommandEx:

```
Private IOrderCommandEx orderExecutor = null;
```

Since the initialization of IOrderCommandEx requires binding an Algorithm, let's create a sample algorithm used in IOrderCommandEx initialization.
Let's start by developing first a Quantity Slicer Algorithm.

Create a class, perhaps named QuantitySliceX1, inheriting from QX.Blitz.Core.QuantityDerivationAlgo. Implement the virtual method GetOrderQuantityToSend, which calculates the slicing quantity for a new order cycle. This algorithm's objective is to divide the total order quantity into manageable chunks to achieve a better average execution price and avoid impacting the market with large orders.
For instance, let's design a simple slicing algorithm that randomly selects an order lot size between 5 and 10 for each sliced order. If the remaining fill quantity is less than 5 lots, the algorithm sends all remaining quantity to ensure a complete order fill on the last round.

```
using System;

using QX.Base.Common;
using QX.Base.Common.Message;
using QX.Blitz.Core;

namespace QX.Blitz.StrategyVada.BreakoutX1
{
    sealed class QuantitySliceAlgoX1 : QuantityDerivationAlgo
    {
        public QuantitySliceAlgoX1(long instrumentID, string ivObjectName)
            : base(instrumentID, ivObjectName)
        {
        }

        public override string GetName()
        {
            return "QuantitySliceX1";
        }

        public override void GetOrderQuantityToSend(IVInfo ivInfo, QuantityDerivationEventArgs eventArgs)
        {
            int orderLotQuantity = new Random().Next(5, 10);
            if(eventArgs.FilledQuantity < eventArgs.OriginalQuantity)
            {
                int actualPendingOrderQty = eventArgs.OriginalQuantity - eventArgs.FilledQuantity;
                int orderQtyToSend = Math.Min(orderLotQuantity* ivInfo.IVInstrument.Instrument.LotSize, actualPendingOrderQty);
                eventArgs.NewOrderQuantity = orderQtyToSend;
                eventArgs.OrderFlag = true;
            }
        }
    }
```

The method GetOrderQuantityToSend is invoked with every change in the market price of the Instrument to which SmartOrderCommandEx is mapped.

We now have a one simple order slicing algorithm.  Lets start working on price quoting algorithm. The price quoting algorithm decide the price at which the slicing quantity to be hit. 

We device here again one simple Pricing Algorithm that first quote order for first 2 seconds on its own side by remain at the best price and if after 2 second we still have some position left from our main order, lets quote bit more aggressively for next 1 second in mid of best bid and ask and beyond 3 seconds from the time of first order initiation if we still have any remaining order, quote the price at market by crossing the spread.

Below is sample class named PriceFinderAlgoX1 representing our quoting algorithm and the class is  inherited from QX.Blitz.Core.PriceDerivationAlgo. Implement an override method GetOrderPriceAndValidity to set a order quoting price of sliced order.

To indicate you are accepting to send an order, you mark OrderFlag as  true
eventArgs.OrderFlag = true;

With every slicing the order price may be new 

```
sealed class PriceFinderAlgoX1 : PriceDerivationAlgo
    {
        private DateTime _instructionDateTime = DateTime.Now;

        public PriceFinderAlgoX1(long instrumentID, string ivObjectName)
            : base(instrumentID, ivObjectName)
        {

        }

        public override string GetName()
        {
            return "PriceFinderAlgoX1";
        }

        public override void GetOrderPriceAndValidity(IVInfo ivInfo, PriceDerivationEventArgs eventArgs)
        {
            TimeSpan ts = DateTime.Now - eventArgs.InstructionSetTime;

            eventArgs.NewValidity = TimeInForce.GFD;

            if (eventArgs.TargetOrderSide == OrderSide.Buy)
            {
                if (ts.TotalSeconds < 2)
                {
                    if (eventArgs.CurrentOrderInfo != null)
                        eventArgs.NewPrice = ivInfo.MarketDataContainer.GetBestBiddingPrice(eventArgs.CurrentOrderInfo);
                    else
                        eventArgs.NewPrice = LastReceivedMarketDataEvent.TouchLineInfo.BestBidPrice + LastReceivedMarketDataEvent.InstrumentInfo.TickSize;
                }
                else if (ts.TotalSeconds >= 2 && ts.TotalSeconds < 3)
                {
                    eventArgs.NewPrice = (ivInfo.MarketDataContainer.BestBidPrice + ivInfo.MarketDataContainer.BestAskPrice) / 2.0;
                }
                else
                {
                    eventArgs.NewPrice = ivInfo.MarketDataContainer.BestAskPrice  -  0.05;
                }
                eventArgs.OrderFlag = true;
            }
            else
                eventArgs.OrderFlag = false;

            }
            else if (eventArgs.TargetOrderSide == OrderSide.Sell)
            {
                if (ts.TotalSeconds < 2)
                {
                    if (eventArgs.CurrentOrderInfo != null)
                        eventArgs.NewPrice = ivInfo.MarketDataContainer.GetBestBiddingPrice(eventArgs.CurrentOrderInfo);
                    else
                        eventArgs.NewPrice = LastReceivedMarketDataEvent.TouchLineInfo.BestAskPrice - LastReceivedMarketDataEvent.InstrumentInfo.TickSize;
                }
                else if (ts.TotalSeconds >= 2 && ts.TotalSeconds < 3)
                {
                    eventArgs.NewPrice = (ivInfo.MarketDataContainer.BestBidPrice + ivInfo.MarketDataContainer.BestAskPrice) / 2.0;
                }
                else
                {
                    eventArgs.NewPrice = ivInfo.MarketDataContainer.BestBidPrice  +  0.05;
                }
                eventArgs.OrderFlag = true;
            }
            else
                eventArgs.OrderFlag = false;

        }
    }
```

Once you have created the QuantitySliceAlgoX1 and PriceFinderAlgoX1 classes in your strategy development project, you can integrate them into your OrderCommandEx mechanism as follows:

```
IOrderCommandEx _orderCommandEx = null;

protected override void OnInitialize()
{
    _ivInfo = base.GetIVInfo(_ivInstrument);
    _orderCommandEx = this.GetSmartOrderExecutor("MomentumStrategyX1" + _ivInfo.IVInstrument.Instrument.DisplayName, _ivInstrument);

     _orderCommandEx.SetPriceDerivationAlgo(new PriceFinderAlgoX1(_ivInfo.IVInstrument.InstrumentID, ivInfo.IVObject.Name));
     _orderCommandEx.SetQuantityDerivationAlgo(new QuantitySliceX1(ivInfo.IVInstrument.InstrumentID, ivInfo.IVObject.Name);

     _orderCommandEx.SetOrderQuantity(InputOrderLotQuantity * orderCommandEx.IVInfo.LotSize);
}
```

As shown in the code snippet above, you instantiate your IOrderCommandEx object within the OnInitialize method using the GetSmartOrderExecutor method. Then, you attach both your order slicer and pricing algorithms using the SetPriceDerivationAlgo and SetQuantityDerivationAlgo methods respectively.

You can use the SetOrderQuantity method to specify the maximum order quantity when the order signal is triggered.

Now, whenever you decide to execute a long position at any point in your strategy logic, you simply call:

```
_ orderCommandEx.SetIgnoreRefLimitPriceFlag(true);
_ orderCommandEx.EnterLong(0);
```

The EnterLong instruction is managed by the system with the assistance of your defined execution algorithms, facilitating the seamless attainment of the desired position.

Following are Execution command supported by the IOrderCommandEx

EnterLong: EnterLong is an instruction in trading that allows a trader to establish a long position in the market according to the specified order quantity. If there is already an existing long position in the same quantity as specified, this instruction will be disregarded, preventing duplicate orders and ensuring that the position remains consistent with the trader's intentions

EnterShort: EnterShort is a trading command used to initiate a short position in the market based on the specified order quantity. If there is already an existing short position in the same quantity as specified, this instruction will be ignored to prevent redundant orders and maintain the desired position in alignment with the trader's strategy.

SetPosition: SetPosition command allows traders to achieve their desired market position, whether it's long or short, by specifying the quantity of assets they wish to trade. A negative quantity represents a short position, indicating the intention to sell, while a positive value signifies a long position, indicating the desire to buy. Setting the quantity to 0 instructs the system to square off all existing positions, effectively closing out any open positions in the market.

GoFlat: GoFlat is a command used in trading to swiftly close out all open positions in the market. It effectively squares off any existing positions, regardless of whether they are long or short, ensuring that the trader has no active exposure to the market. This command is particularly useful for managing risk or when the trader wants to exit all positions quickly, such as at the end of a trading session or in response to unexpected market events.


Here are some additional important methods of the IOrderCommandEx interface along with their relevance:

void SetIgnoreRefLimitPriceFlag(bool value)
The function SetIgnoreRefLimitPriceFlag allows the specification of a benchmark price when executing commands like EnterLong orEnterShort. When set to true, this indicates that a benchmark price must be provided as an argument to the EnterLong command. This benchmark price serves as a reference point for the execution algorithm, enabling it to optimize the execution logic based on the user's intention.

```
_orderCommandExBankNifty.SetIgnoreRefLimitPriceFlag(false);
_orderCommandExBankNifty.EnterLong(_orderCommandExBankNifty.IVInfo.MarketDataContainer.LastPrice);

_orderCommandExNifty.SetIgnoreRefLimitPriceFlag(false);
_orderCommandExNifty.EnterShort(_orderCommandExNifty.IVInfo.MarketDataContainer.LastPrice 0);
```

Conversely, when SetIgnoreRefLimitPriceFlag is set to false, a value of 0 is typically passed in the EnterLong command (or similar commands). In this scenario, the execution logic proceeds without considering any specific benchmark provided by the user, allowing for more generalized execution strategies.

```
_orderCommandExBankNifty.SetIgnoreRefLimitPriceFlag(true);
_orderCommandExBankNifty.EnterLong(0);

_orderCommandExNifty.SetIgnoreRefLimitPriceFlag(true);
_orderCommandExNifty.EnterShort(0);
```

#### UserText Property

The UserText property allows developers to include customized text with any order initiated through the active execution command. This text is then associated with the respective order and execution message, appearing in the Trading dashboard for easy identification and tracking.

For example, consider the scenario of executing a straddle strategy using call and put options: 

```
callOptionsOrderCommandEx.SetIgnoreRefLimitPriceFlag(true);
callOptionsOrderCommandEx.UserText = "ExecuteStraddle";
callOptionsOrderCommandEx.SetPosition(DummyIgnorePrice, -1 * Input_OrderLotQty * callOptionsOrderCommandEx.IVInfo.LotSize);

putOptionsOrderCommandEx.SetIgnoreRefLimitPriceFlag(true);
putOptionsOrderCommandEx.UserText = "ExecuteStraddle";
putOptionsOrderCommandEx.SetPosition(DummyIgnorePrice, -1 * OrderLotQty * putOptionsOrderCommandEx.IVInfo.LotSize);
```

In this example, both the call options and put options orders are tagged with the same user-defined text, "ExecuteStraddle". This enables traders to easily identify and track orders related to the execution of the straddle strategy within the Trading dashboard.

#### Instruction Type Property

This property hold the current active command enum state. If for given orderCommandEx, user has initiated EnterLong command and system still in the execution phase to attain a desired position. The InstryctionTupe hold a enumn value  OrderCommandExInstructionType.EnterLong

The possible InstructionType state are:

```
enum OrderCommandExInstructionType
{
        None,
        EnterLong,
        EnterShort,
        ExitLong,
        ExitShort,
        IncreaseLong,
        IncreaseShort,
        DecreaseLong,
        DecreaseShort,
        GoFlat,
        SetPosition,
}
```

The InstructionType properties are sometime used to hold certain decision if there is any active command in process.

```
// First check if current options iterator is OTM and the price is in favour by user defined Input_OTM_ProfitPercentage
                if (Input_OTM_ProfitPercentage > 0 &&
                    itmOptionsExecutor.MainOptionsOrderCommandEx.IVInfo.MarketDataContainer.LastPrice < averageTradedPrice)
                {
                    double percentChange = 
                        (averageTradedPrice - itmOptionsExecutor.MainOptionsOrderCommandEx.IVInfo.MarketDataContainer.LastPrice) * 100 / averageTradedPrice;
                    if (percentChange > Input_OTM_ProfitPercentage)
                    {
                        // Shift -1 Strike in case of call and +1 Strike in case of Put

                        IOrderCommandEx orderCommandExecutor = GetOTMOrderCommandExToShift(itmOptionsExecutor.MainOptionsOrderCommandEx);
                        if (orderCommandExecutor == null || orderCommandExecutor.InstructionType != OrderCommandExInstructionType.None)
                            return false;

                        int newPositionShift = itmOptionsExecutor.MainOptionsOrderCommandEx.IVInfo.Statistics.NetPosition;
                        if (newPositionShift >= 0)
                            return false;

                        int currentPosition = orderCommandExecutor.IVInfo.Statistics.NetPosition;
                        if (currentPosition == 0)
                            newPositionShift = -1 * Math.Abs(newPositionShift);
                        else if (currentPosition < 0)
                            newPositionShift = -1 * (Math.Abs(currentPosition) + Math.Abs(newPositionShift));
                        else
                            return false;

                        TraceLogInfo(string.Format("ProcessOTM profit Hit. CurrentOptions: {0}, ShiftToOptions: {1}, LTP: {2}, ATP: {3}, %Change: {4}",
                            itmOptionsExecutor.MainOptionsOrderCommandEx.IVInfo.InstrumentName,
                            orderCommandExecutor.IVInfo.InstrumentName,
                            itmOptionsExecutor.MainOptionsOrderCommandEx.IVInfo.MarketDataContainer.LastPrice, averageTradedPrice, percentChange));

                        itmOptionsExecutor.MainOptionsOrderCommandEx.UserText = string.Format("OTMProfitBookHit. LTP: {0}, ATP: {1}, %Change: {2}",
                            itmOptionsExecutor.MainOptionsOrderCommandEx.IVInfo.MarketDataContainer.LastPrice, averageTradedPrice, percentChange);
                        itmOptionsExecutor.MainOptionsOrderCommandEx.GoFlat();

                        orderCommandExecutor.UserText = string.Format("OTM profit Sifting. FromInstrument: {0}",
                                                                                 itmOptionsExecutor.MainOptionsOrderCommandEx.IVInfo.InstrumentName);
                        orderCommandExecutor.SetPosition(0, newPositionShift);
                        return true;
                    }
                }
                return false;
            }
```