<!-- <html>
<head></head>

<body style="background-color: blue;"> -->

## BlitzTrader Developer Guide
<!-- <h1 style="color:Red;">BlitzTrader Developer Guide</h1> -->

Welcome to the .NET programmer’s guide for Blitz Trading Strategy development with C#.
### Introduction 
BlitzTrader empowers developers with a comprehensive framework to build trading strategies effortlessly, catering to a wide spectrum of complexity, from simplest to the most intricate. Powered by the BlitzTrader .NET framework, developers can swiftly and effectively build trading strategies tailored for Blitz. This guide aims to acquaint you with the process of developing BlitzTrader .NET strategies, which operate within the BlitzTrader server application container. 

> The BlitzTrader API was built using Microsoft® .Net 4.7.1 technology. The BlitzTrader event architecture decouples Strategy development from the trading platform,  ensuring that the most critical events are alerted to your Strategy in real-time for informed decision-making.

The Blitz SDK is available for the .NET framework 4.7.1 or above. It comprises a suite of libraries and tools meticulously designed to streamline and accelerate trading strategy development for .NET developers. Whether crafting strategies for personal trading endeavors or catering to institutions such as proprietary trading firms or seasoned traders, the Blitz SDK equips developers with convenient classes and frameworks.

BlitzTrader stands out for its commitment to streamlining the development process of trading strategies to empower the entire algorithmic trading industry. Through this initiative, BlitzTrader not only enhances the efficiency and consistency of strategy creation but also fosters a community where best practices are shared and innovation thrives.  Streamlining the strategy creation process is at the core of BlitzTrader.

The BlitzTrader library provides convenient classes and frameworks that handle communication with the Blitz trading platform, market data handling, order state management, dashboard communication, and seamless connectivity and communication with OMS and exchanges. 

The Blitz Strategy development API is an extensive API that enables strategies to focus on core trading logic while managing heavy lifting by itself. There are no limitations, and you can develop strategies of any complexity across asset classes and multiple markets.

The core features of the API include:
- Managing input and output communication with the trading dashboard and strategy directly from strategy code variables
- Manage the Instrument using market neutral concept called IVObject 
- Accessing exchange-provided real-time market data in your strategy.
- Accessing historical data and any third-party market information
- Integrating with external systems for information or data.
- Accessing over 100+ indicators and providing a framework to build custom indicators.
- Developing high-frequency to mid-frequency strategy models across multiple markets
- Developing and debugging complex options execution strategy models with ease.
- Providing a native order API, Smart Order API, and Smart Order Executor API to handle complex order state flows with a single line of code.
- Providing callback functions for market data and order state change events.
- Provides an IVInfo concept to access all instrument data points that can be associated with an IVObject i.e Instrument properties like name, lot size, Strike Price etc. Market Data properties like Bid Price, Ask Price, LTP etc, Instrument Statistics like NetPosition, MTM, etc.
<!-- </body>
</html> -->