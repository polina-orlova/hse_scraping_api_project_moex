# News sentiment and stock market reaction

**HSE Advanced Programming 2025 \- 26 | Group Project: Data Engineering**

**Team:** Orlova Polina, Shklyaeva Varvara, Bezruchko Maria, Grebenkina Elizaveta, Stelmak Yaroslav, Stepanova Daria

## What this project is about

We collect news articles from RBC about 9 major Russian public companies and combine them with historical stock prices from the Moscow Exchange. The goal is to study whether news sentiment is related to stock price movement the next day.

**Business question:** Does the tone of an RBC article about a company predict how its stock price moves in the next 24 hours?

**Business description:** Russian public companies are among the most actively covered topics in Russian business media. RBC \- one of the largest financial news portals in Russia \- publishes hundreds of articles per week about major exchange-listed companies. At the same time, retail and institutional investors actively trade these stocks on the Moscow Exchange.  
A natural question for any market participant is: does media tone actually move stock prices  
or does the market ignore news and price in information through other channels? Answering this  
question requires a large, clean, matched dataset of news and prices \- which is exactly what  
this project builds.

**Hypotheses:**  
**Н1:** Negative news reduces the price more than positive news raises it.  
**H2:** On the day the news was released, the trading volume was above average.  
**H3:** News published after 18:00 has a stronger effect on the next day's stock price.

## Data sources

**Source 1 \- MOEX ISS API** (API collection)

Official API of the Moscow Exchange. We collect historical daily stock prices and additional information for 9 companies: SBER, GAZP, LKOH, ROSN, MTSS, GMKN, MGNT, ALRS, VTBR. The period is January 2018 \- December 2025\. Authentication is done via login and password through passport.moex.com. We make 5 different types of API requests per company (candles, security info, dividends, market data, index membership) \- 45 requests in total.

**Source 2 \- RBC Scraping** (web scraping)

News articles from rbc.ru about the same 9 companies. We use Selenium to load JavaScript-rendered pages and BeautifulSoup to extract article text. Text is cleaned using regular expressions. Sentiment is scored using the seara/rubert-tiny2-russian-sentiment model from Hugging Face \- a Russian BERT-based model that outputs positive, negative and neutral scores for each article.

## Repository structure

notebooks/  in branch: feature/api

    01\_moex\_api\_SBER\_final\_2018\_2025.ipynb \- MOEX API data collection, Sberbank

    02\_moex\_api\_GAZP\_2018\_2025.ipynb \- MOEX API data collection, Gazprom

    03\_moex\_api\_LKOH\_2018\_2025.ipynb \- MOEX API data collection, Lukoil

    04\_moex\_api\_ROSN\_2018\_2025.ipynb \- MOEX API data collection, Rosneft

    05\_moex\_api\_MTSS\_2018\_2025.ipynb \- MOEX API data collection, MTS

    06\_moex\_api\_GMKN\_2018\_2025.ipynb \- MOEX API data collection, Nornickel

    07\_moex\_api\_MGNT\_2018\_2025.ipynb \- MOEX API data collection, Magnit

    08\_moex\_api\_ALRS\_2018\_2025.ipynb \- MOEX API data collection, Alrosa

    09\_moex\_api\_VTBR\_2018\_2025.ipynb \- MOEX API data collection, VTB

    10\_export\_csvs\_45.ipynb \- export all collected data to 45 CSV files

    11\_combine\_csvs.ipynb \- merge all CSVs into one MOEX dataset

notebooks/ in branch: feature/scraping

    f\_pars\_sber.ipynb \- RBC scraping, Sberbank

    f\_pars\_gazp.ipynb \- RBC scraping, Gazprom

    f\_pars\_lkoh.ipynb \- RBC scraping, Lukoil

    f\_pars\_rosn.ipynb \- RBC scraping, Rosneft

    f\_pars\_mts.ipynb \- RBC scraping, MTS

    f\_pars\_gmkn.ipynb \- RBC scraping, Nornickel

    f\_pars\_mgnt.ipynb \- RBC scraping, Magnit

    f\_pars\_alrs.ipynb \- RBC scraping, Alrosa

    f\_pars\_vtb.ipynb \- RBC scraping, VTB

    all\_data\_dataset\_creation\_final.ipynb \- combine all scraped data \+ sentiment scoring

notebooks/  in branch: feature/eda

    vsemerg.ipynb \- merge MOEX prices with RBC news → final dataset

eda/  in branch: feature/eda

    EDA\_analysis\_final.ipynb \- exploratory data analysis

parametrs.txt \- logging configuration (level, filename, on/off)  
requirements.txt  
.gitignore  
README.md

## Final dataset

**Link to data:** [https://drive.google.com/drive/folders/1YagR4mU9fDBlHhRay81-SPX7KbqFDlU5?usp=sharing](https://drive.google.com/drive/folders/1YagR4mU9fDBlHhRay81-SPX7KbqFDlU5?usp=sharing) 

The final dataset is produced by vsemerg.ipynb. It joins each news article with the stock price on the day of publication and the price on the next trading day. The difference between these two prices is the target variable.

### General statistics

| Number of rows | after cleaning 9,243 > 10000 while parsing|
| :---- | :---- |
| Number of columns | 16 |
| Companies | 9 |
| Period | 2018-01-01 \- 2025-12-31 |

### Column descriptions

| Column | Type | Description |
| :---- | :---- | :---- |
| ticker | string | Company ticker (SBER, GAZP, LKOH, ROSN, MTSS, GMKN, MGNT, ALRS, VTBR) |
| published\_at | datetime | Exact date and time of article publication on RBC |
| date\_trading | date | Trading day linked to the article. If the article was published on a weekend, the date is shifted to the next Monday because the exchange does not work on weekends |
| title | string | Article headline |
| text | string | Full article body text |
| category | string | RBC topic category  |
| url | string | Link to the original article on rbc.ru |
| sentiment\_positive | float 0–1 | Positive sentiment score from seara/rubert-tiny2-russian-sentiment |
| sentiment\_negative | float 0–1 | Negative sentiment score from seara/rubert-tiny2-russian-sentiment |
| sentiment\_neutral | float 0–1 | Neutral sentiment score from seara/rubert-tiny2-russian-sentiment |
| price\_today | float | Closing price of the stock on the trading day of the article |
| price\_next | float | Closing price of the stock on the next trading day |
| is\_dividend\_day | int (0 or 1\) | 1 if this day is a dividend record date. On such days the price drops by roughly the dividend amount \- this is not caused by news |
| in\_imoex | int (0 or 1\) | 1 if the ticker is a member of the IMOEX index |
| target\_pct | float | (target variable) Price change in %: (price\_next − price\_today) / price\_today \* 100 |
| volume | int  | Trading volume (number of shares traded) on the trading day |

### Missing values

| Column | Missing rows | Share | What it means |
| :---- | :---- | :---- | :---- |
| text | 994 | \~10.8% | Articles without body text (short announcements or redirect pages on RBC). Filled with NaN |
| category | \~120 | \~1.3% | Uncategorized articles. Filled with NaN |
| sentiment\_positive, sentiment\_negative, sentiment\_neutral | 994 | \~10.8% | Sentiment could not be computed because text is missing. Filled with NaN |
| All other columns | 0 | 0% | No missing values |

### Other notes on the data

- **Weekend shift:** 1,038 articles were published on Saturday or Sunday. Their date\_trading is shifted to the following Monday so that a stock price always exists for that date.  
- **Dividend days:** 45 rows are marked with is\_dividend\_day \= 1\. These should be treated carefully in analysis because the price drop on such days is caused by the dividend payment, not by news.  
- **Target distribution:** the target variable is approximately centered at zero. The average value after positive news is \+0.42%, after negative news it is −0.38%.  
- **All 9 tickers are in IMOEX**, so in\_imoex \= 1 for all rows. The column is kept for completeness and for potential extension of the dataset with non-index companies.  
- This is a regression task because the target variable is continuous.

## Gitflow

We follow gitflow during development:

- main \- stable version only  
- feature/api \- MOEX API notebooks  
- feature/scraping \- RBC scraping notebooks  
- feature/eda \- merge notebook and EDA notebook

