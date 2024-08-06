## Strategy Instance

A Strategy Instance is a programmable component integrated and configured within the BlitzTrader platform. It functions akin to an object instantiated from a class during runtime. Its usability is contingent upon the system administrator assigning permissions and roles to trading users to utilize Strategies. 

Subsequently, it becomes the trading user's responsibility to curate a functional portfolio of strategies by allocating tradable instruments and configuring strategy parameters. This process of portfolio creation and parameter assignment is referred to as the Strategy Instance creation process.

BlitzTrader retains the Strategy Instance and preserves its state, including trade statistics. The lifecycle of a Strategy Instance persists until all associated instruments remain active or are explicitly deleted. In many intraday trading scenarios, traders prefer to initiate a fresh Strategy Instance every new trading day to ensure clean trade statistics and optimal performance.
