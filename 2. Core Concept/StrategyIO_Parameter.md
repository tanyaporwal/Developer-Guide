### Strategy Input and Output Parameter

The algorithmic strategy often requires variable states that can be controlled or modified from the trading dashboard, or relayed its latest value to external monitoring systems.

By tagging class variables with the StrategyParameter attribute, you enable them to be modified externally, providing flexibility and customization options for your strategy. Adjust the parameters and their default values according to your strategy's requirements such as order trade quantities or stop-loss limits etc.
In the Blitz .NET SDK, Strategy developers can designate certain internal variables of their Strategy class as input variables. These variables can then be modified or controlled by traders via the trading dashboard. Input variables are defined by placing the StrategyParameter attribute above the respective variable declarations.
Below is a code snippet from a sample strategy, where some global variables of the strategy class are tagged with the StrategyParameter attribute:

![Input Parameter](https://github.com/pcnetworking/Markdown/blob/main/images/Strategy%20Input%20and%20output%20parameter.png?raw=true)

All the variables tagged with StrategyParameter attribute will be available to trader dashboard for modification. The StrategyParameter constructor takes following input.

```
[StrategyParameterAttribute("SLLegPercentage", DefaultValue = 0, Description = "Stop Loss Percentage.", CategoryName = "Risk", DisplayName = "Stop Loss Percentage")]
private double Input_SLLegPercentage = 0;
```
In this snippet:

-	The Input_SLLegPercentage variable is tagged with the StrategyParameter attribute.
-	The StrategyParameter constructor is used to provide metadata for the input variable, including its name, default value, description, category name, and display name.
-	Once tagged with the StrategyParameter attribute, the variable becomes available for modification via the trader dashboard.

| Field Name | Data Type | Description |
| :--- | :--- | :--- |
| Name | String | The identifier used to denote a variable. The same name is appears in the UI dashboard for a tagged variable |
| Default Value | Object | A default value of a input variable displayed in the trader dashboard. It holds all primitive data type define in C# including String. |
| Description | String | The description text used to describe the variable. It will be displayed in the trader dashboard for corresponding  name. |
| Category Name | String | The various inputs are tagged under some logical user defined category. The input variable will be displayed in the property grid under the given category group name |

The trading dashboard allows to change any variable value of type input variable for given strategy instance register in server container.

**The Output Parameter**

Use output parameters to relay important information or calculated results from your strategy to the trading dashboard, providing traders with valuable insights and data visualization.

For example, some variables within the strategy might calculate proprietary information, and it's desirable to display the final result on the trading dashboard.

To achieve this, variables within the strategy class can be tagged with the StrategyPublishParameter attribute. Any change in the value of a variable tagged with this attribute will be relayed in realtime in the trading dashboard.

Below is an example code snippet showcasing the usage of StrategyPublishParameter attribute:

[StrategyPublishParameter("EntryRound", DefaultValue = 0, Description = "Entry Round")]
private int Output_EntryRound = 0;

In this snippet:
-	The Output_EntryRound variable is tagged with the StrategyPublishParameter attribute.
-	The StrategyPublishParameter constructor is used to provide metadata for the output variable, including its name, default value, and description.
-	Once tagged with the StrategyPublishParameter attribute, any change in the value of Output_EntryRound will be automatically displayed on the trading dashboard, enabling real-time monitoring of the strategy's output.
