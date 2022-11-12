# R-Markowitz-Model
This Code optimizes the strategic asset allocation using the Markowitz model

The **modern portfolio theory** (MPT) is a practical method for selecting investments in order to maximize their overall returns within an acceptable level of risk. This mathematical framework is used to build a portfolio of investments that maximize the amount of expected return for the collective given level of risk.

American economist Harry Markowitz pioneered this theory in his paper "Portfolio Selection," which was published in the Journal of Finance in 1952.
He was later awarded a Nobel Prize for his work on modern portfolio theory.

A key component of the MPT theory is diversification. Most investments are either high risk and high return or low risk and low return. Markowitz argued that investors could **achieve their best results by choosing an optimal mix of the two based on an assessment of their individual tolerance to risk**.

<sub>https://www.investopedia.com/terms/m/modernportfoliotheory.asp</sub>

## Overview
- [Financial data collection](#1-financial-data-collection)
- [Mean-variance analysis](#2-mean-variance-analysis)

## 1 Financial data collection
First, the script collects financial data from the **Yahoo’s finance website**. The user can choose the stocks, the time frame, the number of stocks to be included, and how to handle missing values.
> Since Yahoo was bought by Verizon, there have been several changes with their API. They may decide to stop providing stock prices in the future. So the method used in this code may not work in the future.

### 1.1 R packages to download stock price data
The most popular way to get financial data into R is the quantmod package:
```R
install.packages("quantmod")
```
The prices downloaded in by using quantmod are xts zoo objects. For our calculations we will use tidyquant package which downloads prices in a tidy format as a tibble. You can download the tidyquant package by typing:
```R
install.packages("tidyquant")
```
> tidyquant includes quantmod so you can install just tidyquant and get the quantmod packages as well.

## 2 Mean-variance analysis
### 2.1 Mean returns and covariance
We begin the mean-variance analysis of the user’s chosen stocks by computing the mean returns and the covariance matrix.

# Done!
![myImage](https://media.giphy.com/media/XRB1uf2F9bGOA/giphy.gif)

### Sources:
https://www.codingfinance.com/post/2018-03-27-download-price/
