### SDK Architecture

![imgxt](https://github.com/pcnetworking/Markdown/blob/main/introduction.png?raw=true)

As shown in the illustration, the programmable strategy is loaded in Blitz Server Strategy component container and provides an interface of communication with internal OMS, RMS, Market Data Feed server and Trading Dashboard. The Blitz Server has all the capabilities to connect with any exchanges or broker OMS APIs for both order routing and market data information.

The following step-by-step example is done with Microsoft Visual Studio 2017 or newer version
1. In main menu,  select File->New->Project… to create a new project
2. In New Project wizard, select Visual C#->Class Library (.NET Framework). Input the project Name MyFirstStrategyX1 in Name field, the press OK.


![add new project](https://github.com/pcnetworking/Markdown/blob/main/making%20project.png?raw=true)

After the project is created, the source editor will appear. The initial step is to add a reference to the Blitz SDK into the project.

Following are the main assembly that must be referenced in your strategy project.
From your VS project you add a local reference of Blitz framework provided DLL’s. In the Solution Explorer, right-click References, and select Add Reference.

In the Reference Manager, select Browse and then navigate to the directory where the Blitz Assemblies are located: **<INSTALL_DIR>\Bin\QX.Blitz\..**
Select and add following named DLL, then click OK.

After the project is created, the source editor will appear. The initial step is to add a reference to the Blitz SDK into the project.

Following are the main assembly that must be referenced in your strategy project.
From your VS project you add a local reference of Blitz framework provided DLL’s. In the Solution Explorer, right-click References, and select Add Reference.

In the Reference Manager, select Browse and then navigate to the directory where the Blitz Assemblies are located: **<INSTALL_DIR>\Bin\QX.Blitz\..**
Select and add following named DLL, then click OK.

| Assembly | Description |
| :------- | :---------- |
| QX.Base.Common.dll | Definded Enums and common structure used in projects |
| QX.Base.Core.dll | Provides the access of Strategy Base class and related functionality
| QX.Base.Data.dll |
| QX.Base.Financial.dll | Provides functionality to access options pricing libraries including Greeks |
| QX.Common.Lib.dll | Helper Library |
| QX.Common.Helper.dll | Helper Library |

Refer following DLL in case you need to use TimeDataSeries and Indicator library framework 
| Assembly | Description |
| :------- | :---------- |
| QX.FinLib.dll | Technical Indicator, TimeDataSeries |

Below is a generic class template for the Strategy class, MainStrategy, within your first project, MyFirstStrategyX1. It features various override method sections for handling different events. The DLL generated from your programmable Strategy and its dependencies must be placed in the Blitz installation folder located at **<INSTALL_DIR>\Bin\Strategies**. For details overview of Blitz Setup, you refer to Blitz Setup Guide.

BlitzServer will automatically detect your Strategy DLL. Through the administrative process, you can load and  assign the Strategy to a set of trader accounts, enabling the Strategy to run in simulated or live setups as needed.

### C# Code Block
```
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Linq;
using System.Runtime.CompilerServices;
using System.Text;
using System.Threading;
using System.Timers;
using System.Data;

using QX.Base.Common;
using QX.Base.Common.InstrumentInfo;
using QX.Base.Common.Message;
using QX.Base.Core;
using QX.Base.Data;
using QX.Blitz.Core;
using QX.Blitz.Core.Series;
using QX.Common.Helper;
using QX.Common.Lib;

using QX.FinLib;
using QX.FinLib.Data;
using QX.FinLib.Common.Util;
using QX.FinLibDB;
using QX.Blitz.Core.InstrumentInfo;

namespace QX.Blitz.Strategy.PaitTrading
{
    [StrategyAttribute("{AD672C4A-3BB9-4070-A605-B9AD63F677F6}", "QX.PairTradeAlphaX1", "PairTradeAlphaX1", "X1", "QXT")]
    public class MainStrategy : StrategyBase
    {

        private IVObject _ivInstrumentPair1 = new IVObject("Pair1", "Pair1 Tradable Instrument", true,
                                                           InstrumentType.Equity | InstrumentType.Futures |
                                                           InstrumentType.Options,
                                                           MarketDataType.All, OrderEventType.All);

private IVObject _ivInstrumentPair2 = new IVObject("Pair2", "Pair2 Tradable Instrument", true,
                                                          InstrumentType.Equity | InstrumentType.Futures |
                                                          InstrumentType.Options,
                                                          MarketDataType.All, OrderEventType.All);

        private IVInfo _ivInfoPair1 = null;

        private IVInfo _ivInfoPair2 = null;


        #region Input Parameters

        [StrategyParameterAttribute("ClientID", 1, "ClientID")]
        private string Input_ClientID = "1";

        [StrategyParameterAttribute("BarSize", 1, "Minute Bar Size")]
        private int Input_MinBarSize = 1;

        [StrategyParameterAttribute("OrderQtyLots", 1, "Order Quantity in Lots")]
        private int Input_OrderQtyLots = 1;

        [StrategyParameterAttribute("SLPercentage", 2, "Stop Loss")]
        private double Input_SLPercentage = 2;

        [StrategyParameterAttribute("PTPercentage", 5, "Profit Target")]
        private double Input_PTLPercentage = 5;

        [StrategyParameterAttribute("SquareOffTime", "15:20", "SquareOffTime")]
        private string Input_SquareOffTime = "15:20";


        #endregion

        #region Published Parameters

        [StrategyPublishParameter("LastState", DefaultValue = "", Description = "Last State", DisplayName = "Last State")]
        private string Output_LastState = "X";

        [StrategyPublishParameter("MarketSpread", DefaultValue = 0, Description = "Market Spread", DisplayName = "Market Spread")]
        private double Output_MarketSpread = 0;

        [StrategyPublishParameter("MTM", DefaultValue = 0, Description = "MTM")]
        private double Output_MTM = 0;

        [StrategyPublishParameter("TransactionCount", DefaultValue = 0, Description = "TransactionCount")]
        private int Output_TransactionCount = 0;

        #endregion


        private IOrderCommandEx _orderCommandPair1Ex = null;
        private IOrderCommandEx _orderCommandPair2Ex = null;

        private TimeDataSeries _timeDataSeriesFutures = null;
        private QX.FinLib.Data.IndicatorBase _superTrendIndicator = null;

        private Dictionary<long, IOrderCommandEx> _orderExecutionMAP = new Dictionary<long, IOrderCommandEx>();

        protected override void OnInitialize()
        {

            _ivInfoPair1 = base.GetIVInfo(_ivInstrumentPair1);
            _ivInfoPair2 = base.GetIVInfo(_ivInstrumentPair2);

            TraceLogInfo("Strategy Initialize Successfully");
        }

        protected override bool OnStart(out string errorString)
        {
            try
            {
                errorString = string.Empty;

                return true;

            }
            catch (Exception exp)
            {
                errorString = exp.Message;


                TraceLogError(string.Format("Error in Start: {0} ", exp.Message));
                return false;
            }
        }

        protected override void OnStopping()
        {
            base.SaveStrategySettings("TransactionCount", Output_TransactionCount);

            foreach (IOrderCommandEx orderCommandEx in _orderExecutionMAP.Values)
            {
                if (orderCommandEx.IVInfo.Statistics.NetPosition != 0)
                {
                    orderCommandEx.UserText = string.Format("SquareOffPositionInvokedAtStrategyStop");
                    orderCommandEx.GoFlat();
                }
            }

            TraceLogInfo("Strategy Stopped.");
        }

        public override Version BuildVersion
        {
            get
            {
                return new Version(1, 05, 09, 09);
            }
        }

        public override IVObject[] IVObjects
        {
            get
            {
                List<IVObject> _ivObjectArray = new List<IVObject>();
                _ivObjectArray.Add(_ivInstrumentPair1);
                _ivObjectArray.Add(_ivInstrumentPair2);

                return _ivObjectArray.ToArray();
            }
        }

        protected override int MarketDataDelayNotifyTimeInSeconds
        {
            get { return 10; }
        }

        public override bool RequiredToPublishParameterInStrategyStopMode
        {
            get
            {
                return true;
            }
        }

        public override bool RequiredToSendMarketDataInStrategyStopMode
        {
            get
            {
                return true;
            }
        }

        public override bool ValidateInstrumentAssignments(KeyValuePair<IVObject, Instrument>[] ivObjetToInstrumentMap, out string errorString)
        {
            errorString = string.Empty;

            return true;
        }

        protected override bool ValidateStrategyParameter(string parameterName, object paramterValue, out string errorString)
        {
            errorString = string.Empty;
            bool retValue = false;

            switch (parameterName)
            {
                default:
                    retValue = base.ValidateStrategyParameter(parameterName, paramterValue, out errorString);
                    break;
            }
            return retValue;
        }

        protected override void OnStrategyParamterChanged()
        {

        }


        protected override ActionCommandInfo[] GetActionCommands()
        {
            List<ActionCommandInfo> actionCommandList = new List<ActionCommandInfo>();

            actionCommandList.Add(GetGoFlatActionCommand());

            return actionCommandList.ToArray();
        }

        private static readonly Guid GoFlatCommandStaticID = new Guid("{4775106D-E7FA-4538-9CD9-304B1BBEE954}");

        private ActionCommandInfo GetGoFlatActionCommand()
        {
            List<ActionCommandFieldInfo> actionCommandParameterInfoList = new List<ActionCommandFieldInfo>();
            return CreateActionCommand(GoFlatCommandStaticID, "Go Flat", false, actionCommandParameterInfoList.ToArray());
        }

        protected override void ExecuteActionCommand(Guid commandStaticID, ActionCommandFieldInfo[] inputFields)
        {
            base.ExecuteActionCommand(commandStaticID, inputFields);

            if (GoFlatCommandStaticID.Equals(commandStaticID))
            {
                foreach (IOrderCommandEx orderCommandEx in _orderExecutionMAP.Values)
                {
                    if (orderCommandEx.IVInfo.Statistics.NetPosition != 0)
                    {
                        orderCommandEx.UserText = string.Format("ManualSquareOff");
                        orderCommandEx.GoFlat();
                    }
                }
            }
        }

        protected override void OnStrategyStatusQuery()
        {
            TraceLogInfo(string.Format("OnStrategyStatusQuery: Leg1 Position: {0}, Leg2 Position: {1}",
                _ivInfoPair1.Statistics.NetPosition, _ivInfoPair2.Statistics.NetPosition));
        }

        protected override void OnDispose()
        {
            if (!IsInitialized)
                return;

            TraceLogInfo("Strategy Disposing");
        }


        protected override void OnMarketDataEvent(StrategyMarketDataEventArgs eventArgs)
        {

        }

        protected override void OnTrade(IVObject ivObject, TradeDataEventArgs eventArgs)
        {

        }

        protected override void OnOrderAccepted(IVObject ivObject, OrderAcceptedEventArgs orderAcceptedEventArgs)
        {
        }

        protected override void OnOrderRejected(IVObject ivObject, OrderRejectedEventArgs orderRejectedEventArgs)
        {
        }

        protected override void OnOrderModificationRejected(IVObject ivObject, OrderModificationRejectEventArgs orderModificationRejectEventArgs)
        {

        }

        protected override void OnOrderCancellationRejected(IVObject ivObject, OrderCancellationRejectEventArgs orderCancellationRejectEventArgs)
        {
        }

        protected override void OnOrderCancelled(IVObject ivObject, OrderCancellationAcceptedEventArgs orderCancellationAcceptedEventArgs)
        {
        }

        protected override void OnOrderModified(IVObject ivObject, OrderModificationAcceptedEventArgs orderModificationAcceptedEventArgs)
        {
        }

    }

}
```
In the upcoming sections, we will delve into the intricacies of the Strategy development process.