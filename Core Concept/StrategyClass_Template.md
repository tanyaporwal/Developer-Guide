### Strategy Class Template

The BlitzTrader strategy is a typical class in your C# project of type.NET Class and inheriting from the provided framework class QX.Blitz.Core.StrategyBase

```
[StrategyAttribute("{611A7560-1353-4852-A916-1F5FC19277F2}", "XT.PairTradingX1", "Pair TradingStrategy", "X1", "QXT")]
public class MainStrategy : StrategyBase
{
}
```

Your strategy class MainStrategy implements some necessary overrides method of base class like OnInitialize(), OnStart(), OnStop(), OnMarketDataEvent(), OnTrade() etc. to define how your strategy interacts with the Blitz algorithmic trading platform. Customize these methods with your specific logic to create a functional trading strategy. Save this code in a file named MainStrategy.cs within your C# project.

As you see, the class is decorated with the Strategy attribute.  The StrategyAttribute decorates the class, providing essential metadata such as a unique identifier (GUID), name, description, version, and owner of the strategy to the BlitzTrader platform. Adjust the attribute parameters according to your strategy's specifications.

Your strategy development project may consist of multiple classes and dependencies on third-party components or external systems. However, it must include one primary class serving as the main strategy class, inheriting from StrategyBase. This main strategy class is crucial as it defines the core behavior and interactions of your strategy with the BlitzTrader platform.

The following code structure of your strategy with file name MainStrategy.cs:

```
using QX.Base.Common;
using QX.Base.Common.InstrumentInfo;
using QX.Base.Common.Message;
using QX.Base.Core;
using QX.Base.Data;
using QX.Blitz.Core;
using QX.Blitz.Core.Series;
using QX.Common.Helper;

[StrategyAttribute("{611A7560-1353-4852-A916-1F5FC19277F2}", "XT.PairTradingX1", "Pair TradingStrategy", "X1", "QXT")]
public class MainStrategy : StrategyBase
{
    // 1.   Define your Strategy static IVObject variables (Statically or Dynamically)
    // 2.   Define strategy Input and Output variables
    // 3.   Define strategy local variables as per your strategy logic
    // 4.   Define and Implements strategy Overrides methods
    // 5.   Use framework provided order execution and helper functions to build your 
               Strategy alpha and execution logics
}
```

Within your main strategy class, you have the flexibility to define various elements essential for your strategy's functionality including IVObject variables, strategy input and output variables, local variables, and override methods to implement your strategy logic.  Leveraging the functionalities provided by the BlitzTrader framework is essential for building your strategy's logic. These functionalities include order execution methods, market data and order event handling mechanisms, and more. Utilize these tools to implement your strategy's alpha generation and execution logic effectively.

