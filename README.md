# Fincancial Planning Tools
This project showcases planning tools available to credit union customers to help them evaluate their financial health. Two specific areas are highlighted in this project: 1- helping customers evaluate their monthly budgets to include an emergency fund and 2- helping them forecast a reasonably effective strategy for their retirement based on their current holdings of stocks, and bonds.

This program is a protype for demonstration purposes that utilizes pandas data analysis and visualiztions, as well as a Monte Carlo Simultion to evaluate the expected performance of financial holdings over the next 10-30 years.

---

## Technologies

Python implementation: CPython

Python version       : 3.7.13

IPython version      : 7.31.1


---

## Packages & Libraries

import os

import requests

import json

import pandas as pd

from dotenv import load_dotenv

import alpaca_trade_api as tradeapi

from MCForecastTools import MCSimulation

%matplotlib inline`

---

## Part 1: Evaluating Financial Health: Gathering Data

***examples shown are fictious*** 

1. **Values of crypotcurrency holdings are pulled into a Jupyter Notebook using an API call to Free Crypto API, then the requests and Json libraries are used to translate the data to a dictionary, which is sliced to find the price data for analysis.**
    
![code image api_call](./images/request_library.png)

    - slicing the response object to access the current prices of BTC and ETH

`btc_price = btc_response["data"]["1"]["quotes"]["USD"]["price"]`

`eth_price = eth_response["data"]["1027"]["quotes"]["USD"]["price"]`

    - current crypto prices are multiplied by the holdings amount and added togther to give a total value for the customer's crypto wallet
  
`btc_value = btc_price * btc_coins`

`eth_value = eth_price * eth_coins`

**`total_crypto_wallet = btc_value + eth_value`**


2. **Values of Stocks and Bond holdings are pulled into a Jupyter Notebook using Alpaca SDK API, then the get_bars function of the Alpaca trade api are used to translate the data to a dictionary, which is sliced to find the required data for analysis.**

    - .env file is created to hold Alpaca API key and secret key, then read with get function from the os library

`alpaca_api_key = os.getenv("ALPACA_API_KEY")`

`alpaca_secret_key = os.getenv("ALPACA_SECRET_KEY")`

    - Alpaca trade API REST object and parameters are indicated to call the data
    
`alpaca = tradeapi.REST(alpaca_api_key, alpaca_secret_key, api_version="v2")`
    
    - a dataframe called response_df is set up to hold the data called using the get_bars funcion with pre-established parameter variables
    
`response_df = alpaca.get_bars(tickers, timeframe, start=start_date, end=end_date).df`
    
    - the data in the data frame was cleaned and organized to gather the latest closing price for the customer's financial holdings, then the values were saved as variables
    

```
SPY = response_df[response_df['symbol']=="SPY"].drop('symbol', axis=1)
AGG = response_df[response_df['symbol']=="AGG"].drop('symbol', axis=1)
portfolio_df = pd.concat([SPY,AGG], axis=1, keys=["SPY", "AGG"])
agg_close_price = portfolio_df.loc[start_date, ("AGG", "close")].astype(float)
spy_close_price = portfolio_df.loc[start_date, ("SPY", "close")].astype(float)
```

    -finally the latest closing price for the holdings was multiplied by the number of shares held, then added together to find the total value of the stocks and bonds portfolio, with the total printed out for the customer.
**`total_portfolio = total_crypto_wallet + total_stocks_bonds`**

`print(f"The current value of your cryptocurrency wallet is ${total_crypto_wallet: .2f}")`

## Evaluating Financial Health: Data Analysis for establishing an Emergency Fund

1. **The total financial holdings established above were manipulated and reframed into a new dataframe using Pandas, and then visualized into a pie graph.**

![wallet_total_df](./images/wallet_df.png)

![portfolio_pie](./images/portfolio_pie.png)

2. **Python code was used to mathmatically determine if the customer's portfolio value and stated monthly income is enough to allow for the establishment of an Emergency Fund. The results were filtered with 3 conditions to give the customer a response stating their financial health.**

`emergency_fund_value = 3 * monthly_income`
``` 
if total_portfolio > emergency_fund_value:
    print("Congratulations! You have enough money in your portfolio for an emergency fund.")
elif total_portfolio == emergency_fund_value:
    print("Congratultations on meeting this important financial goal.")
else:
    print(f"You are ${total_portfolio - emergency_fund_value} away from reaching your goal of having enough money for an emergency fund")
```

---

## Part 2: Evaluating Financial Health: Retirement Savings
1. **In this section, we used the MCForecastTools library to create a Monte Carlo simulation for the member’s retirement savings portfolio to look forward 30 years then 10 years**

    - Gather 3 years of historical data on the customer's current Stock and Bond holdings via an API call through Alpaca. The same methods were used to gather the data as in Part 1, with start_date and end_date expanded to 3 years.
    
```    
start_date = pd.Timestamp("2019-10-19", tz="America/New_York").isoformat()
end_date = pd.Timestamp("2022-10-19", tz="America/New_York").isoformat()
prices_df = alpaca.get_bars(tickers, timeframe, start=start_date, end=end_date).df
```
    
    - The data was cleaned and reorganized as seen in Part 1, and saved into the `prices_df` dataframe.


2. **The 3 years worth of clean data was read into the Monte Carlo Simulation, for a 30 year outlook**

    - 500 simulation runs where executed with a 60/40 weighted bias stock(SPY) to bond(AGG) for 30 years of trading days
    
`MC_30yr = MCSimulation(portfolio_data = prices_df, weights = [.60,.40], num_simulation = 500, num_trading_days = 252*30)`

    
    - The cumulative returns were calculated in the Simulation and plotted
    
   `MC_30yr.calc_cumulative_return()`
   
   `MC_sim_line_plot = MC_30yr.plot_simulation()`

![Monte_Carlo_plot](./images/MC_plot.png) 

    - The probablility distribution was calculated in the Simulation and plotted
    
    `MC_sim_dist_plot = MC_30yr.plot_distribution()`

![distribution_plot](./images/MC_distr_plot.png)

    - Finally, the 30 year simulation cumulative return totals were summarized for general statistics which were used to find the dollar value of 95% confidnece interval for future returns

![MC_30yr_summary_stats](./images/30_sum_stats.png)

# Use the lower and upper `95%` confidence intervals to calculate the range of the possible outcomes for the current stock/bond portfolio
`ci_lower_thirty_cumulative_return = MC_30yr_summary_stats[8] * total_stocks_bonds`
`ci_upper_thirty_cumulative_return = MC_30yr_summary_stats[9] * total_stocks_bonds`

**Print the result of your calculations**
`print(f"Over the next 30 years, there is a 95% chance that the current bond portfolio value will end in returns between ${ci_lower_thirty_cumulative_return: .2f} and ${ci_upper_thirty_cumulative_return: .2f}")`

    
3. **The Monte Carlo Simulation, was repeated for a 10 year outlook to determine if early retirement was an option**

    - All the above steps in 1 & 2 were repeated with 2 changes: stocks(SPY) weighted to .80/ bonds(AGG) to .20, and the timeframe for the 500 simulated runs was changed to 10 years of trading days
    
    `MC_10yr = MCSimulation(portfolio_data = prices_df, weights = [.80,.20], num_simulation = 500, num_trading_days = 252*10)`
    
    - All calculations and plots were repeated to end with the simulation's cumulative return totals summarized into general statistics then used to find the dollar value of 95% confidnece interval for future returns
    
![MC_10yr_summary_stats](./images/10_sum_stats.png)

# Use the lower and upper `95%` confidence intervals to calculate the range of the possible outcomes for the current stock/bond portfolio
`ci_lower_ten_cumulative_return = MC_10yr_summary_stats[8] * total_stocks_bonds`
`ci_upper_ten_cumulative_return = MC_10yr_summary_stats[9] * total_stocks_bonds`

**Print the result of your calculations**
`print(f"Over the next 10 years, there is a 95% chance that the current bond portfolio value will end in returns between ${ci_lower_ten_cumulative_return: .2f} and ${ci_upper_ten_cumulative_return: .2f}")`

----

## Analysis Conclusion
The protoype shown above is a simple python code using API data collection, Pandas dataframe manipulation, and Monte Carlo Simulation model outlooks to allow credit union customers a quick view into their financial health. 

Analysis of the value of current holdings shows whether a cushion is avaialbe for an emergency fund, or if continued investment is necessary.

Future retirement outlooks using past performance of their holdings forcasted into future possibilities gives an average expected return shedding light on time vs. retirement savings for different investment strategies. 

---

## Contributors

This project was in conjunction with UC Berkeley staff and myself Jodi Artman.  *artman.jodi@gmail.com*

---

## License

licensed in accordance with UC Berkeley policy
