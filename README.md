[.](<a href="https://imgur.com/MpkfU6j"><img src="https://i.imgur.com/MpkfU6j.png" title="source: imgur.com" /></a>)

 - This is a comparison of the housing market and the stock market from a high level looking at volatility as well as returns over the last decade 

## The U.S. Housing market according to the Case Shiller Index provided by the FED looking at 20 cities ##

```
cities = ['PHXRNSA','ATXRSA','BOXRSA','NYXRSA','DAXRSA','SEXRNSA','CHXRSA','MIXRNSA','POXRSA','CEXRSA','WDXRSA','DNXRSA', 'CSUSHPISA']
dfs = []
for city in cities:
    city_url = f"https://fred.stlouisfed.org/graph/fredgraph.csv?bgcolor=%23e1e9f0&chart_type=line&drp=0&fo=open%20sans&graph_bgcolor=%23ffffff&height=450&mode=fred&recession_bars=on&txtcolor=%23444444&ts=12&tts=12&width=1168&nt=0&thu=0&trc=0&show_legend=yes&show_axis_titles=yes&show_tooltip=yes&id={city}&scale=left&cosd=2010-01-01&coed=2020-05-01&line_color=%234572a7&link_values=false&line_style=solid&mark_type=none&mw=3&lw=2&ost=-99999&oet=99999&mma=0&fml=a&fq=Monthly&fam=avg&fgst=lin&fgsnd=2020-02-01&line_index=1&transformation=lin&vintage_date=2020-08-15&revision_date=2020-08-15&nd=1987-01-01"
    df = pd.read_csv(city_url, index_col='DATE')
    dfs.append(df)
df = pd.concat(dfs, axis=1)
df.rename(columns={'PHXRNSA': 'Phoenix', 'ATXRSA': 'Atlanta', 'BOXRSA': 'Boston', 'NYXRSA': 'New York', 'DAXRSA': 'Dallas', 'SEXRNSA': 'Seattle', 'CHXRSA': 'Chicago', 'MIXRNSA': 'Miami', 'POXRSA': 'Portland', 'CEXRSA': 'Cleveland', 'WDXRSA': 'Washington D.C.', 'DNXRSA': 'Denver', 'CSUSHPISA': 'U.S.A'}, inplace=True)
df
```
[.](<a href="https://imgur.com/RPHsLql"><img src="https://i.imgur.com/RPHsLql.jpg" title="source: imgur.com" /></a>)

## In Order to make an Apples to Apples comparison we must convert the data from FRED to a percent change, the dates not alligning correctly will be dealt with below ##

```
usa_housing_market = pd.DataFrame(df, columns=['U.S.A'])
usa_house_pct = usa_housing_market.pct_change()
usa_house_pct.reset_index(inplace = True)
usa_house_pct.head()
```
[.](<a href="https://imgur.com/uFtkZfm"><img src="https://i.imgur.com/uFtkZfm.jpg" title="source: imgur.com" /></a>)

## Pulling in the stock market (SPY) information with the alapacas API ##
- The tricky part is having the data reflect the average of a month and post it to a day at the at the end for comparison against housing data.

```
ticker = ["SPY"]
timeframe = "1D"
start_date = pd.Timestamp('2010-01-01', tz='America/New_York').isoformat()
end_date = pd.Timestamp('2020-05-01', tz='America/New_York').isoformat()

df_spy = api.get_barset(
    ticker,
    timeframe,
    limit=None,
    start=start_date,
    end=end_date,
    after=None,
    until=None,
).df

# Only comparing closing value
df_spy = df_spy.drop(
    columns=['open', 'high', 'low', 'volume'],
    level=1
)
#Allows the data to reflect the average of a month and post it to a day at the at the end for comparison against housing data.
df_spy.index = df_spy.index.date
df_spy.reset_index(inplace = True)
df_spy.rename(columns={'index': 'Date'}, inplace= True)
dfspy = df_spy.set_index('Date')
dfspy.index = pd.to_datetime(dfspy.index)
monthly_spy = dfspy['SPY'].resample('M').mean()
monthly_spy
```
[.](<a href="https://imgur.com/eWxJOyj"><img src="https://i.imgur.com/eWxJOyj.jpg" title="source: imgur.com" /></a>)


## Bringing the two markets together to asses the markets against each other. ##

```
Housing_vs_Stocks = pd.concat([usa_house_pct.reset_index(drop=True),spy_pct.reset_index(drop=True)], axis=1)
Housing_vs_Stocks = Housing_vs_Stocks.set_index('DATE')
Housing_vs_Stocks.rename(columns={'U.S.A.':'US Housing Maket','close': 'Stock Index'}, inplace= True)
Housing_vs_Stocks.head()
```
[.](<a href="https://imgur.com/Q9MU5ZK"><img src="https://i.imgur.com/Q9MU5ZK.jpg" title="source: imgur.com" /></a>)

## Which one is more volatile? ##

```
cumulative_returns = (1 + Housing_vs_Stocks).cumprod()
cumulative_returns.hvplot(y=['U.S.A', 'Stock Index'],width = (1000), height = (500), xticks = (10),
             value_label='Observed Cumulative Returns')
```
[.](<a href="https://imgur.com/y4JgTV1"><img src="https://i.imgur.com/y4JgTV1.jpg" title="source: imgur.com" /></a>)

# In this project we brought together our Python skills, along with APIs, EDA, Visualization and Comparison techniques to demonstrate a collection of techniques learned so far in our bootcamp #
