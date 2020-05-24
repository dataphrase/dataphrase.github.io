---
title: "Risk and Returns: The Sharpe Ratio📊"
date: 2020-05-16
tags: [Python, Flask, Finance, Statistics]
header:
  image: "/images/sharpe_ratio_blog/sharpe_blog.jpg"
excerpt: "Calculating and comparing profitability and risk of different investments using the Sharpe Ratio."
---

## Introduction💡
Reward-to-variability ratio, also known as The Sharpe Ratio, is one of the most popular risk/return measures in finance. It helped professor William Sharpe win a Nobel Prize in Economics in 1990 for his work on the capital asset pricing model (CAPM). The Sharpe Ratio is usually calculated for a portfolio and uses the risk-free interest rate as benchmark. But, today we'll learn about the Sharpe ratio by calculating it for the stocks of the two tech giants Facebook and Amazon. As benchmark we'll use the S&P 500 that measures the performance of the 500 largest stocks in the US.

## Why S&P 500?🤔
When we use a stock index instead of risk-free rate, the result is called the Information Ratio and is used to benchmark the return on active portfolio management because it tells you how much more return for a given unit of risk your portfolio manager earned relative to just putting your money into a low-cost index fund.

## Visualizing daily prices📈
Before we compare an investment in either Facebook or Amazon with the index of the 500 largest companies in the US, let's visualize the data, so we can understand it in a better way. Let us start by looking at Amazon and Facebook.
<img src="{{ site.url }}{{ site.baseurl }}/images/sharpe_ratio_blog/ama_fb_graph.PNG" alt="Amazon and Facebook Stock data">
Now, let's take a closer look at S&P 500, our benchmark.
<img src="{{ site.url }}{{ site.baseurl }}/images/sharpe_ratio_blog/sp_500_graph.PNG" alt="S&P 500 data">

## Inputs for Sharpe Ratio📨
The Sharpe Ratio uses difference in returns between the two investment opportunities under consideration. However, our data shows the historical value of each investment, not the return, hence we have to calculate the returns. Let's start with daily stock returns. We can calculate that as follows -
```python
    stock_returns = stock_data.pct_change()
```
This is the result after plotting the stock returns -
<img src="{{ site.url }}{{ site.baseurl }}/images/sharpe_ratio_blog/stock_returns.PNG" alt="Percentage change in stocks">
In the same way, we need to find daily S&P 500 returns.
```python
    sp_returns = benchmark_data['S&P 500'].pct_change()
```
This is what you get after plotting the returns -
<img src="{{ site.url }}{{ site.baseurl }}/images/sharpe_ratio_blog/sp_500_returns.PNG" alt="Percentage change in S&P 500">

Next, we need to calculate relative performance of stocks vs. the S&P 500 benchmark.
```python
    # axis = 0 should be specified for row-wise subtraction
    excess_returns = stock_returns.sub(sp_returns, axis=0)
```
After plotting excess returns -
<img src="{{ site.url }}{{ site.baseurl }}/images/sharpe_ratio_blog/excess_returns.PNG" alt="Excess Returns">

## Calculating the Sharpe Ratio📝
The first step is to calculate the average difference in daily returns, stocks vs S&P 500. For this, we can simply find the mean of excess returns.
```python
    avg_excess_return = excess_returns.mean()
```
After plotting -
<img src="{{ site.url }}{{ site.baseurl }}/images/sharpe_ratio_blog/mean_returns.PNG" alt="Mean Returns">

Next step is to find the standard deviation of the return difference. This shows us the amount of risk an investment in the stock implies as compared to an investment in the S&P 500.
```python
    sd_excess_return = excess_returns.std()
```
After plotting this on a bar chart -
<img src="{{ site.url }}{{ site.baseurl }}/images/sharpe_ratio_blog/sd_returns.PNG" alt="Standard Deviation of Returns">

Now, we just need to compute the ratio of average excess returns and standard deviation of excess returns, whose result is the Sharpe Ratio and indicates how much more (or less) return the investment opportunity under consideration yields per unit risk.

This ratio is often annualized by multiplying it to the square root of the number of periods. We have used daily data as input so we will use the square root of the number of trading days (5 days, 52 weeks, minus a few holidays) : √252

```python
    daily_sharpe_ratio = avg_excess_return.div(sd_excess_return)

    annual_factor = np.sqrt(252)

    annual_sharpe_ratio = daily_sharpe_ratio.mul(annual_factor)
```
Finally, here is the result we have been waiting for...
<img src="{{ site.url }}{{ site.baseurl }}/images/sharpe_ratio_blog/result.PNG" alt="Result">

## Conclusion💸
The difference was mostly driven by differences in return rather than risk between Amazon and Facebook. The risk of choosing Amazon over Facebook (as measured by standard deviation) was only slightly higher so that the higher Sharpe ratio for Amazon ends up higher mainly due to the higher average daily returns for Amazon.

So, which investment should we go for? In 2016, Amazon had a Sharpe ratio twice as high as Facebook. This means that an investment in Amazon returned twice as much compared to the S&P 500 for each unit of risk an investor would have assumed. In other words, in risk-adjusted-terms, the investment in Amazon would have been more attractive.

## Sharpe Ratio Calculator💻
This is a web app which can tell you which stock to invest in, based on the Sharpe Ratio. Currently I have used 8 different stocks, namely Microsoft, Google, Facebook, Genpact, Tesla, Advanced Micro Devices, Intel and Apple. I will try my best to expand to more companies, but for now, this is it. You can access the web app by clicking [here](https://sharpe-ratio-calculator.herokuapp.com/).
Alternatively, you can see the web app in live action by watching this video👇
<video width="450" height="310" controls>
  <source src="{{ site.url }}{{ site.baseurl }}/videos/sharpe_ratio_calc.mp4" type="video/mp4">
</video>
