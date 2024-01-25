# Analyst-Reports-NLP

An university research project on Natural Language Processing (NLP), Data Mining and Financial Analysis.


## Table of contents
- [Introduction](#introduction)
- [Data Sources](#data-sources)
  - [Main Data](#main-data)
  - [Enrichment Data](#enrichment-data)
- [Motivation and Objective](#motivation-and-objective)
- [Approach](#approach)
  - [Data Acquisition](#data-acquisition) 
  - [Data Merge](#data-merge)
  - [Data Analysis](#data-analysis)
  - [Model Building and Benchmarking](#model-building-and-benchmarking)
- [Results and Conclusion](#results-and-conclusion)


# Introduction
In this project, we obtain financial research containing both, a report in natural language and a quantifiable prediction on the underlying asset. We further analyze the predictions and analyze the texts and data in order to improve the analysts accuracy - our benchmark. To do so we build basic metrics regarding the texts, but also classification models based on large language models (LLMs). We focus on the (subjective) analyst note and (objective) bull and bear arguments. We can show that the analysts forecasts are merely based on chance. And by leveraging the classifications of our fine-tuned LLM, and the confidence in its predictions, our model outperforms the analysts steadily and shows significant predictive qualities -- while using the analysts own words.

# Data Sources
There are two data sources used in this project:
1. [Morningstar Equity Research](https://www.morningstar.co.uk/uk/research/equities)
1. [Yahoo Finance Python API (unofficial)](https://pypi.org/project/yfinance/)

## Main Data
As our main data we utilize the Morningstar Equity Research containing both natural language and a quantifiable prediction. A more in depth description of the gathered variables can be found [here](Data/README.md). Currently, there are ~1600 reports, where some are pseudo-duplicates (same company, traded on different exchanges).

## Enrichment Data
To better analyze the financial reports and identify the analysts accuracy, we utilize yahoo finance as enrichment data. We pull the OHLCV (Open High Low Close Volume) data on all different companies.

# Motivation and Objective

>"There are three ways to make a living in this business: be first; be smarter; or cheat. Now, I don't cheat. And although I like to think we have some pretty smart people in this building, it sure is a hell of a lot easier to just be first." - John Tuld, fictional character in 'Margin Call' (2011)

But to be first - in this analogy of margin call - you need to either be located on the stock exchange and have the fastest connection or you have to create your own signal. This is why we try to find a unique pattern of hidden information in the text data.


# Approach
## Data Acquisition
We design a parser [1. Data Acquisition](https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/e2b421149b506df3004cbe21ee8ec53f33352a56/1.%20Data%20Acquisition.ipynb) based on manual created cookies and parse a list of all available reports. 

We use the ```requests``` package to parse the landing page of all [reports](https://www.morningstar.co.uk/uk/research/equities/page/1?PageSize=2000) and save the table and underlying ```URL_ID```.

To access the underlying links we generate them by concatenating them from partial links:
```
url1 =  'https://tools.morningstar.co.uk/ukp/stockreport/default.aspx?Site=uk&id='

url2 = '&tab=15&isreport=true&LanguageId=en-GB&SecurityToken='

url3 ='%5D3%5D0%5DE0WWE$$ALL'

url = url1 + df["URL_ID"][i] + url2 + df["URL_ID"][i] + url3  

```

Since the reports are behind a login secured webpage, we either have to use a headless browser like ```selenium``` to completely automate the process or we use a semi-automatic approach and create cookies for the security handshake through manual login:

1. Login with credentials on [Morningstar](https://www.morningstar.co.uk/)
1. Navigate to [List of equity reports](https://www.morningstar.co.uk/uk/research/equities) 
1. Activate network on browser in developer view 
1. Click on any item in List of reports
1. Search for get GET package to that site and right-click ```copy as cURL```
1. Paste on website [curlconverter](https://curlconverter.com/python/) to get cookies and headers for Python ```request``` package
1. Paste below the cookies and headers

The result of this approach will have this form:

```
cookies = {
    'PSI': 'S',
    'RT_uk_BS': '+I7qdl0ruLBRlX6zrB8CNg==',
    'RT_uk_CD': 'efNq51LXeHZjftQvvSJlDw==',
    'RT_uk_GI': 'mGr7p+708bJo/rsmM3mC2saJLu1zqcp3rB+tAgdYcgcaeDBhQnzEInIY9N50/4tl',
    'RT_uk_MD': 'PO5Mf2z103h9gY8gAJ+EDQ==',
    'RT_uk_MS': '8X3mhE0/kf6o/dIJeMo7TA==',
    'RT_uk_PS': '6G2wEmwHr8HiGZATAp96Mw==',
    'ad-profile': '%7b%22AudienceType%22%3a21%2c%22UserType%22%3a2%2c%22PortofolioCreated%22%3a0%2c%22IsForObsr%22%3afalse%2c%22NeedRefresh%22%3atrue%2c%22NeedPopupAudienceBackfill%22%3afalse%2c%22EnableInvestmentInUK%22%3a-1%7d',
    'RT_uk_LANG': 'en-GB',
}

headers = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Sec-Fetch-Site': 'same-site',
    # 'Cookie': 'PSI=S; RT_uk_BS=+I7qdl0ruLBRlX6zrB8CNg==; RT_uk_CD=efNq51LXeHZjftQvvSJlDw==; RT_uk_GI=mGr7p+708bJo/rsmM3mC2saJLu1zqcp3rB+tAgdYcgcaeDBhQnzEInIY9N50/4tl; RT_uk_MD=PO5Mf2z103h9gY8gAJ+EDQ==; RT_uk_MS=8X3mhE0/kf6o/dIJeMo7TA==; RT_uk_PS=6G2wEmwHr8HiGZATAp96Mw==; ad-profile=%7b%22AudienceType%22%3a21%2c%22UserType%22%3a2%2c%22PortofolioCreated%22%3a0%2c%22IsForObsr%22%3afalse%2c%22NeedRefresh%22%3atrue%2c%22NeedPopupAudienceBackfill%22%3afalse%2c%22EnableInvestmentInUK%22%3a-1%7d; RT_uk_LANG=en-GB',
    'Sec-Fetch-Dest': 'document',
    # 'Accept-Encoding': 'gzip, deflate, br',
    'Sec-Fetch-Mode': 'navigate',
    'Host': 'tools.morningstar.co.uk',
    'Accept-Language': 'en-gb',
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/16.6 Safari/605.1.15',
    'Referer': 'https://www.morningstar.co.uk/',
    'Connection': 'keep-alive',
}
```


## Data Merge
Unfortunately the ```TickerSymbol``` of the analyst report is not specific enough. Thats why the Jupyter Notebook [2. Data Merge](https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/e2b421149b506df3004cbe21ee8ec53f33352a56/2.%20Data%20Merge.ipynb) shows a detailed approach how to find the actual ticker symbols.  E.g.: "Central Japan Railway Co" has Ticker ```9022```, but is only callable by ```9022.T``` (traded at _Tokyo_ exchange)
Our approach:
- Finding the underlying Ticker symbol based on the names (which are definitely clean)
- We do this by scraping Yahoo Finance search results and select the most likely result based on:
    - Highest traded volume (meaning if others think this is *the* company, so do we) 
    - Matching currency
    - Had round about the same (adjusted close) price as the analyst reports state

This works, as yahoo finance only returns results that definitely fit. The additional criteria are necessary since the same stock can be traded at multiple exchanges and also OTC (over the counter, not public). With the additional criteria and some cleaning of the names and currencies we can achieve a high success rate and only lose less than 2% of the results (some of them because the company is delisted and can no longer be traded).

But since the yahoo API has reliability issues, this is executed in batches and the intermediate results are stored in ['TickerLists'](https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/7001526eac8ff0546cb8925c72620b9aac6c098f/Data/Analyst%20Reports/TickerLists).
We can then parse the OHLCV data (Open High Low Close Volume) on all different companies for 30 and 60 days and store them as JSON and CSV [files](https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/5244627ee13270a4965ce6d756ce2d5a4f35ce44/Data/Market%20Data) and also safe the merged data as a CSV [table](https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/b53f1ff07b1db0ab2db573ed40a8d9dcf192a6c2/Data/final_data.csv).

## Data Analysis
The Jupyter Notebook [3. Data Analysis](https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/5244627ee13270a4965ce6d756ce2d5a4f35ce44/3.%20Data%20Analysis.ipynb) shows a preliminary and superficial approach how to design targets (like accuracy) and features (metrics based on text) and their dedicated correlations.


<img src="https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/536c21c931660d818339fa48b3c223f640383bc4/Plots/Bulls_WordCloud.png" alt="isolated" width="380"/> <img src="https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/536c21c931660d818339fa48b3c223f640383bc4/Plots/Bears_WordCloud.png" alt="isolated" width="380"/>

<img src="https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/ff81fa24b619a7ed5ca76bd925821c3d9c9fb1d4/Plots/Bulls_network.png" alt="isolated" width="380"/> <img src="https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/ff81fa24b619a7ed5ca76bd925821c3d9c9fb1d4/Plots/Bears_network.png" alt="isolated" width="380"/> 

<img src="https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/ff81fa24b619a7ed5ca76bd925821c3d9c9fb1d4/Plots/correlation_matrix.png" alt="isolated" width="760"/> 





## Model Building and Benchmarking
The final Jupyter Notebook [4. Model Building](https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/5244627ee13270a4965ce6d756ce2d5a4f35ce44/4.%20Model%20Building.ipynb) show more refined approaches of how to predict the analysts estimate, the actual trend and the authors confidence and use of causal language. 


In order to validate learned models -- here the Bayesian networks -- we compare predictions made from the model with previously separated test data. In this way, we can objectively evaluate them according to their predictive qualities. For this comparison, we utilise these fundamental principles:
\newline \textit{Sensitivity} or true positive rate ($\mathrm{TPR}$) is derived from the true positives $\mathrm{TP}$,  i.e., the correctly identified positives $\mathrm{P}$ from the test set: 
$
\mathrm{TPR}=\frac{\mathrm{TP}}{\mathrm{P}}
$
\newline \textit{Specificity} or true negative rate ($\mathrm{TNR}$) is derived from the true negatives $\mathrm{TN}$,  i.e., the correctly identified negatives $\mathrm{N}$ from the test set: 
$
\mathrm{TNR}=\frac{\mathrm{TN}}{\mathrm{N}}
$
\newline By plotting both the sensitivity and specificity in relation, we obtain the so called Receiver Operating Curve (ROC). For the results of both measures 1 or 100\% is the optimum, and if the curve is the diagonal, we observed a random process.  In Figure~\ref{fig:exroc}, we can see some common examples of curves. In order to further summarise these evaluations, we can calculate the Area Under the Curve (AUC) to rank the models. Again, a value of 0.5 indicates a random process. 

<img src="https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/main/Plots/ROC.png" alt="isolated" width="380"/> 

- Explain methodology for ROC Curves and why they are better (no even distribution etc)

- Explain how the probabilities are used to improve the result and what else one could try in the future (Use the probability tuples of each column trained individually as input of a text classifier and use that as an input to a random forest/XGBoost... and then predict the labels)

- How do we measure the actual and predicted trends (average of 30 or 60d adjusted close) (since sporadic jumps in the market are not relevant for long term investors which are the target group of readers); for trend 6% and for actual 2% to be considered sideways movement


<img src="https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/ff81fa24b619a7ed5ca76bd925821c3d9c9fb1d4/Plots/ROC_curves_prediction.png" alt="isolated" width="380"/> <img src="https://github.com/trashpanda-ai/Analyst-Reports-NLP-/blob/ff81fa24b619a7ed5ca76bd925821c3d9c9fb1d4/Plots/ROC_curves_actual.png" alt="isolated" width="380"/> 

# Results and Conclusion
We were able to obtain interesting insights in the structure and thought process behind the reports and could also benchmark the analysts. We showed that one can predict the actual stock trend better than the analysts themselves. And also that the analysts come up with very well formulated arguments for or against the stock, but give the wrong recommendation and only mimic the underlying distribution of the current market.