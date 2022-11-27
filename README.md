# Python-Markowitz-Model
This Code optimizes the strategic asset allocation using the Markowitz model

The **modern portfolio theory** (MPT) is a practical method for selecting investments in order to maximize their overall returns within an acceptable level of risk. This mathematical framework is used to build a portfolio of investments that maximize the amount of expected return for the collective given level of risk.

American economist Harry Markowitz pioneered this theory in his paper "Portfolio Selection," which was published in the Journal of Finance in 1952.
He was later awarded a Nobel Prize for his work on modern portfolio theory.

A key component of the MPT theory is diversification. Most investments are either high risk and high return or low risk and low return. Markowitz argued that investors could **achieve their best results by choosing an optimal mix of the two based on an assessment of their individual tolerance to risk**.

<sub>https://www.investopedia.com/terms/m/modernportfoliotheory.asp</sub>

## Overview
0. [Jupyter Notebook](#jupyter-notebook)
1. [Financial data collection](#1-financial-data-collection)
2. [Sharpe Ratio Simulation](#2-sharpe-ratio-simulation)
3. [Closed-form Portfolio Optimization](#3-closed-form-portfolio-optimization)
4. [Advise the User](#4-advise-the-user)

## Jupyter Notebook
As we compiled this program as a jupyter notebook we instruct you to do so as well. Everything you need in order to set up JupyterLab can be found here
> https://jupyter.org/install

Once your jupyter notebook is up and running, you can continue to the next step.

## 1 Financial data collection
First, the script collects financial data from the **Yahoo’s finance website**. The user can choose the amount invested, the time frame and the four tickers of the stocks the user wants to be included.
> Since Yahoo was bought by Verizon, there have been several changes with their API. They may decide to stop providing stock prices in the future. So the method used in this code may not work in the future.

### 1.1 Python to download stock price data
The most popular remote data access for pandas is the pandas-datareader, which works for multiple versions of pandas. Open the terminal on your computer and enter the following command:
```
pip install pandas-datareader
```
Then import the pandas-datareader as well as all libraries for date time support needed to run this code smoothly:
```
from pandas_datareader import data as web
from pandas_datareader._utils import RemoteDataError
from datetime import date, timedelta
import pandas as pd
import sys
```
### 1.2 User Input
In the following sequence multiple while loops are used to take the inputs given by the user. First, the user enters the investment amount he is interested to optimize, then, the user enters a time frame within which the portfolio of the four stocks shall be optimized. The latter are entered last by typing in their ticker (be sure to use the exact same ticker as on Yahoo Finance - whether you use upper- or lowercase letters is irrelevant, however).

```
while True:
    try:
        # user interaction
        # ask for initial capital and ticker
        start_capital = float(input("Enter your initial capital:\n"))
        date_entry = input('Enter begin of testing range in YYYY-MM-DD format:\n')
        year1, month1, day1 = map(int, date_entry.split('-'))
        start_date = date(year1, month1, day1)
        ticker1 = input("Enter your first ticker available at finance.yahoo.com:\n")
        # ask for testing range and convert to date objects
        # data import: read and parse file containing OHLC data into dataframe using pandas datareader
        # strftime is used to convert from datetime to string in specific format wanted by datareader
        data1 = web.DataReader(str(ticker1),
                                   start=start_date.strftime('%Y-%m-%d'),
                                   end=str(date.today()),
                                   data_source='yahoo')
        # convert index column to normal column and add standard numbered index column
        #data1.reset_index(inplace=True, drop=False)
        break
    # error handling for wrong ticker input
    except RemoteDataError:
        print("Error. No information for ticker '{}'. Only enter tickers available on Yahoo Finance.".format(ticker1))
    # error handling for wrong date/starting capital input
    except ValueError:
        print("Error. Enter correct dates and numeric start capital.")

while True:
    try:
        # Use the same structure for the second tickers 
        ticker2 = input("Enter your second ticker available at finance.yahoo.com:\n")
        data2 = web.DataReader(str(ticker2),
                                   start=start_date.strftime('%Y-%m-%d'),
                                   end=str(date.today()),
                                   data_source='yahoo')
        #data2.reset_index(inplace=True, drop=False)
        break
    except RemoteDataError:
        print("Error. No information for your second ticker '{}'. Only enter tickers available on Yahoo Finance.".format(ticker2))
    # error handling for wrong date/starting capital input

while True:
    try:
        # Use the same structure for the third ticker
        ticker3 = input("Enter your third ticker available at finance.yahoo.com:\n")
        data3 = web.DataReader(str(ticker3),
                                   start=start_date.strftime('%Y-%m-%d'),
                                   end=str(date.today()),
                                   data_source='yahoo')
        #data3.reset_index(inplace=True, drop=False)
        break
    except RemoteDataError:
        print("Error. No information for your third ticker '{}'. Only enter tickers available on Yahoo Finance.".format(ticker3))

while True:
    try:
        # uUse the same strucutre for the fourth ticker
        ticker4 = input("Enter your fourth ticker available at finance.yahoo.com:\n")
        data4 = web.DataReader(str(ticker4),
                                   start=start_date.strftime('%Y-%m-%d'),
                                   end=str(date.today()),
                                   data_source='yahoo')
        #data4.reset_index(inplace=True, drop=False)
        break
    except RemoteDataError:
        print("Error. No information for your fourth ticker '{}'. Only enter tickers available on Yahoo Finance.".format(ticker4))
```

### 1.3 Returns
In this step pd.concat() is used to get the closing price of the stocks of interest for each date within the given range.
```
stocks = pd.concat([data1['Close'],data2['Close'],data3['Close'],data4['Close']],axis=1)
stocks.columns = [ticker1,ticker2,ticker3,ticker4]
```
Using this dataframe, one can compute the returns using this return formula:
$$r_t=\displaystyle \frac{p_t-p_{t-1}}{p_{t-1}}$$
Python code ro calculate the return:
```
raw_returns = (stocks-stocks.shift(1))/stocks.shift(1)
```
Then, because log promotes "fairness", calculate the log of returns:
(first import NumPy which offers comprehensive mathematical functions in order to be able to use the log and other mathematical functions)
```
import numpy as np
logReturns = np.log(1+raw_returns)
```

## 2 Sharpe Ratio Simulation

In the following, we determine the Sharpe Ratio (SR) for every portfolio, using this formula:
$$SR(w)=\displaystyle \frac{R(w)-R_f}{σ(w)}$$

By simulating 10000 portfolios, we would be able to tell the user which proportion he should allocate to each stock by finding the best weight factor for each stock. To do so we first define Arrays to then fill with the sharpe ratios of the simulated portfolios. Then we calculate the mean log return, the covariance and in a for loop generate a random starting-weight to then simulate deviating weight factors in order to generate 10000 simulated portfolios.

``` 
numberPortfolios = 10000

weight = np.zeros((numberPortfolios,4))
expectedLogReturn = np.zeros(numberPortfolios)
expectedVol = np.zeros(numberPortfolios)
sharpeRatio = np.zeros(numberPortfolios)

meanLogReturn = logReturns.mean()
sigma = logReturns.cov()
for k in range(numberPortfolios):
    w = np.array(np.random.random(4))
    w = w/np.sum(w)
    weight[k,:] = w
    expectedLogReturn[k]= np.sum(meanLogReturn*w)
    expectedVol[k] = np.sqrt(np.dot(w.T,np.dot(sigma,w)))
    sharpeRatio[k] = expectedLogReturn[k]/expectedVol[k]
```

We can then identify the vector of weights that maximizes the Sharpe Ratio:
```
maxIndex = sharpeRatio.argmax()
weight[maxIndex,:]
```

Next we use Matplotlib, a comprehensive library for creating static, animated, and interactive visualizations in Python, to display the result of our simulation in a scatterplot. 
```
import matplotlib.pyplot as plt
plt.figure(figsize=(20,6))
plt.scatter(expectedVol,expectedLogReturn,c=sharpeRatio)
plt.xlabel('Expected Volatility')
plt.ylabel('Expected Log Returns')
plt.colorbar(label='SR')
plt.scatter(expectedVol[maxIndex],expectedLogReturn[maxIndex],c='red')
plt.title("Identification of the optimal portfolio (simulated version)")
plt.show()
```
![Alt text](/Users/yannickhaenggi/Desktop/HS22/Advanced Programming/Abgabe/simulation-plot.png "Simulated Portfolio Optimization")
<img src="/Users/yannickhaenggi/Desktop/HS22/Advanced Programming/Abgabe/simulation-plot.png" alt="Alt text" title="Optional title">

Below you can find the link to get startet with Matplotlib:
> https://matplotlib.org/stable/users/getting_started/index.html

## 3 Closed-form Portfolio Optimization

After having identified the simulated version of the portfolio, we will complement the analysis with a the closed-form alternative to find the optimal portfolio. We will use this version to advise user, as results are much more stable Efficient Markowitz Frontier.

To do so, we first install the SciPy (pronounced “Sigh Pie”) open-source software for mathematics, science, and engineering.
```
pip install scipy
```

SciPy provides us with functions for minimizing (or maximizing) objective functions, that are possibly subject to constraints.

```
from scipy.optimize import minimize

# a function of weights returning negative SR function
def negativeSR(w):
    w = np.array(w)
    R = np.sum(meanLogReturn*w)
    V = np.sqrt(np.dot(w.T,np.dot(sigma,w)))
    SR = R/V
    return -1*SR

# function that takes in weights and returns zero
def checkSumToOne(w):
    return np.sum(w)-1

# w0 is an "initial guess" which is equal weight for all
w0 = [0.25,0.25,0.25,0.25]
# each wi is between 0 and 1
bounds = ((0,1),(0,1),(0,1),(0,1))
# define type and a function
constraints = ({'type':'eq','fun':checkSumToOne})
w_optimal = minimize(negativeSR,w0,method='SLSQP',bounds=bounds,constraints=constraints)
```

These calculations allow us to obtain the optimal vector that maximizes Sharpe Ratio:
```
opti_w = w_optimal.x
opti_return = sum(opti_w * meanLogReturn)
opti_vol= np.sqrt(np.dot(opti_w.T,np.dot(sigma,opti_w)))
```

Because we use the np.linspace() function, which returns evenly spaced numbers over a specified interval, we quickly calculate the minimal and maximal expected log return to work with the appropriate interval to allow for a nice plot in a next step.
```
minExpectedLogReturn = min(expectedLogReturn)
maxExpectedLogReturn = max(expectedLogReturn)
```

In this step, we can obtain the Efficient Markowitz Frontier (which is located on the boundary of the above scatterplot). For each return (each horizontal line) we plot, we identify the best volatility and then plot the frontier. To do so we use multiple functions which are explained in the comments:
```
returns = np.linspace(minExpectedLogReturn,maxExpectedLogReturn,100)

v_optimal = []

# function that takes in weights and returns volatility
def minimizeVolatility(w):
    w = np.array(w)
    V = np.sqrt(np.dot(w.T,np.dot(sigma,w)))
    return V

def getReturn(w):
    w = np.array(w)
    R = np.sum(meanLogReturn*w)
    return R

for R in returns:
    # find best volatility (as we target minimal volatility we need a constraint on the returns)
    constraints = ({'type':'eq','fun':checkSumToOne},
                  {'type':'eq','fun': lambda w: getReturn(w) - R})
    opt = minimize(minimizeVolatility,w0,method='SLSQP',bounds=bounds,constraints=constraints)
    # save optimal volatility
    v_optimal.append(opt['fun'])
```

Again we use Matplotlib to display the result of our optimization in a scatterplot.
```
import matplotlib.pyplot as plt
plt.figure(figsize=(20,6))
plt.scatter(expectedVol,expectedLogReturn,c=sharpeRatio)
plt.xlabel('Expected Volatility')
plt.ylabel('Expected Log Returns')
plt.colorbar(label='SR')
plt.scatter(expectedVol[maxIndex],expectedLogReturn[maxIndex],c='red')
plt.scatter(opti_vol, opti_return, c="black")
plt.plot(v_optimal,returns,'--')
plt.title("Simulated portfolios and optimal PF based on closed form optimization (black dot)")
plt.show()
```

## 4 Advise the User

Below we use a comprehensive print statement, to tell the user which weight (in %) he should assign to each stock to optimize his portfolio returns. In a next stept, we disclose the exact amount of funds the user shall allocate to each of the four stocks. 
```
print("The optimal weight for {} is {}%, for {} is {}%, for {} is {}% and for {} is {}%.".format(ticker1,round(opti_w[0],2)*100,ticker2,round(opti_w[1],2)*100,ticker3,round(opti_w[2],2)*100,ticker4,round(opti_w[3],2)*100))

print("According to the optimal weights you should invest {}.- in {}, {}.- in {}, {}.- in {} and {}.- isn {}.".format(round(opti_w[0]*start_capital,2),ticker1,round(opti_w[1]*start_capital,2),ticker2,round(opti_w[2]*start_capital,2),ticker3,round(opti_w[3]*start_capital,2),ticker4))
```
In the last step, we use the weights calculated in the Markowitz Optimization to simulate the portfolio performance throughout the timeframe chosen by the user and display a plot to visualize the portfolio performance over the defined timeframe.
```
portfolio_returns = raw_returns.dot(opti_w.T)
portfolio_returns_factors = portfolio_returns + 1
pf_2 = portfolio_returns_factors.dropna()
print(pf_2)

port_ret_cleaned_1= pf_2.cumprod()

chart = port_ret_cleaned_1 * start_capital

plt.figure(figsize=(20,6))
plt.xlabel("Date")
plt.ylabel("Capital")
plt.plot(chart,'--')
plt.title('Capital based on portfolio optimization and timerange estimation',fontsize=12)

plt.show()
```

# Happy Trading!
![myImage](https://media.giphy.com/media/XRB1uf2F9bGOA/giphy.gif)

### Sources:
https://www.codingfinance.com/post/2018-03-27-download-price/
https://www.youtube.com/watch?v=f2BCmQBCwDs
