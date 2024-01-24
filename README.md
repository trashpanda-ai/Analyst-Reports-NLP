# Analyst-Reports-NLP

An university research project on Natural Language Processing (NLP), Data Mining and Financial Analysis.


## Table of contents
- [Introduction](#introduction)
- [Data Sources](#data-sources)
  - [Main Data](#main-data)
  - [Enrichment Data](#enrichment-data)
- [Motivation and Objective](#motivation-and-objective)
- [Approach](#approach)
- [Results and Conclusion](#results-and-conclusion)


# Introduction
In this project, we obtain financial research containing both, a report in natural language and a quantifiable prediction on the underlying asset. We further analyze the predictions and analyze the texts and data in order to improve the analysts accuracy - our benchmark. To do so we build basic metrics regarding the texts, but also classification models based on large language models (LLMs). We focus on the (subjective) analyst note and (objective) bull and bear arguments. We can show that the analysts forecasts are merely based on chance. And by leveraging the classifications of our fine-tuned LLM, and the confidence in its predictions, our model outperforms the analysts steadily and shows significant predictive qualities -- while using the analysts own words.

# Data Sources
There are two data sources used in this project:
1. [Morningstar Equity Research](https://www.morningstar.co.uk/uk/research/equities)
1. [Yahoo Finance Python API (unofficial)](https://pypi.org/project/yfinance/)

## Main Data
As our main data we utilize the Morningstar Equity Research containing both natural language and a quantifiable prediction. A more in depth description of the gathered variables can be found [here](Data/README.md).

## Enrichment Data
To better analyze the financial reports and identify the analysts accuracy, we utilize yahoo finance as enrichment data. We pull the OHLCV (Open High Low Close Volume) data on all different companies.

# Motivation and Objective

>"There are three ways to make a living in this business: be first; be smarter; or cheat. Now, I don't cheat. And although I like to think we have some pretty smart people in this building, it sure is a hell of a lot easier to just be first." - John Tuld, fictional character in 'Margin Call' (2011)

But to be first - in this analogy of margin call - you need to either be located on the stock exchange and have the fastest connection or you have to create your own signal. This is why we try to find a unique pattern of hidden information in the text data.


# Approach
## Data Acquisition
We design a parser [1. Data Acquisition](https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/e2b421149b506df3004cbe21ee8ec53f33352a56/1.%20Data%20Acquisition.ipynb) based on manual created cookies and gather a list of all available reports first. We then parse all reports from Morningstar.
- Explain how data acquisition works and How do we set up the connection via cookie handshake?

## Data Merge
Unfortunately the ```TickerSymbol``` of the analyst report is not specific enough. Thats why the Jupyter Notebook [2. Data Merge](https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/e2b421149b506df3004cbe21ee8ec53f33352a56/2.%20Data%20Merge.ipynb) shows a detailed approach how to find the actual ticker symbols and since the yahoo API has reliability issues, this is executed in batches and the intermediate results are stored in 'TickerLists'.
## Data Analysis
The Jupyter Notebook [3. Data Analysis](https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/5244627ee13270a4965ce6d756ce2d5a4f35ce44/3.%20Data%20Analysis.ipynb) shows a preliminary and superficial approach how to design targets (like accuracy) and features (metrics based on text) and their dedicated correlations.
ToDO:
- Re-run all notebooks with png as export as well

## Model Building and Benchmarking
The final Jupyter Notebook [4. Model Building](https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/5244627ee13270a4965ce6d756ce2d5a4f35ce44/4.%20Model%20Building.ipynb) show more refined approaches of how to predict the analysts estimate, the actual trend and the authors confidence and use of causal language. 
TODO_ 
- Explain methodology for ROC Curves and why they are better (no even distribution etc)
- Explain how the probabilities are used to improve the result and what else one could try in the future (Use the probability tuples of each column trained individually as input of a text classifier and use that as an input to a random forest/XGBoost... and then predict the labels)
- How do we measure the actual and predicted trends (average of 30 or 60d adjusted close) (since sporadic jumps in the market are not relevant for long term investors which are the target group of readers); for trend 6% and for actual 2% to be considered sideways movement

# Results and Conclusion
We were able to obtain interesting insights in the structure and thought process behind the reports and could also benchmark the analysts. We showed that one can predict the actual stock trend better than the analysts themselves. And also that the analysts come up with very well formulated arguments for or against the stock, but give the wrong recommendation and only mimic the underlying distribution of the current market.