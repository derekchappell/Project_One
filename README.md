# Project_One
This is a comparison of the housing market and the stock market from a high level

## The U.S. Housing market according to the Case Shiller Index ##

   cities = ['PHXRNSA','ATXRSA','BOXRSA','NYXRSA','DAXRSA','SEXRNSA','CHXRSA','MIXRNSA','POXRSA','CEXRSA','WDXRSA','DNXRSA', 'CSUSHPISA']
dfs = []
for city in cities:
    city_url = f"https://fred.stlouisfed.org/graph/fredgraph.csv?bgcolor=%23e1e9f0&chart_type=line&drp=0&fo=open%20sans&graph_bgcolor=%23ffffff&height=450&mode=fred&recession_bars=on&txtcolor=%23444444&ts=12&tts=12&width=1168&nt=0&thu=0&trc=0&show_legend=yes&show_axis_titles=yes&show_tooltip=yes&id={city}&scale=left&cosd=2010-01-01&coed=2020-05-01&line_color=%234572a7&link_values=false&line_style=solid&mark_type=none&mw=3&lw=2&ost=-99999&oet=99999&mma=0&fml=a&fq=Monthly&fam=avg&fgst=lin&fgsnd=2020-02-01&line_index=1&transformation=lin&vintage_date=2020-08-15&revision_date=2020-08-15&nd=1987-01-01"
    df = pd.read_csv(city_url, index_col='DATE')
    dfs.append(df)
df = pd.concat(dfs, axis=1)
df.rename(columns={'PHXRNSA': 'Phoenix', 'ATXRSA': 'Atlanta', 'BOXRSA': 'Boston', 'NYXRSA': 'New York', 'DAXRSA': 'Dallas', 'SEXRNSA': 'Seattle', 'CHXRSA': 'Chicago', 'MIXRNSA': 'Miami', 'POXRSA': 'Portland', 'CEXRSA': 'Cleveland', 'WDXRSA': 'Washington D.C.', 'DNXRSA': 'Denver', 'CSUSHPISA': 'U.S.A'}, inplace=True)
df

## Bringing the data together ##

usa_housing_market = pd.DataFrame(df, columns=['U.S.A'])
usa_house_pct = usa_housing_market.pct_change()
usa_house_pct.reset_index(inplace = True)
usa_house_pct.head()

## pulling in the stock market information ##

ticker = ["SPY"]
timeframe = "1D"
start_date = pd.Timestamp('2010-01-01', tz='America/New_York').isoformat()
end_date = pd.Timestamp('2020-05-01', tz='America/New_York').isoformat()

#calling 2010's top ten companies by size ticker
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

## Bringing the two markets together to asses the markets against each other ##

Housing_vs_Stocks = pd.concat([usa_house_pct.reset_index(drop=True),spy_pct.reset_index(drop=True)], axis=1)
Housing_vs_Stocks = Housing_vs_Stocks.set_index('DATE')
Housing_vs_Stocks.rename(columns={'U.S.A.':'US Housing Maket','close': 'Stock Index'}, inplace= True)
Housing_vs_Stocks.head()

## which one is more volatile ##

#From the above graph vlatility appears to be the name of the game for the stock market. Lets double check. 
monthly_std = Housing_vs_Stocks.std()
monthly_std_visule = (1 + monthly_std).cumprod()
monthly_std_visule.hvplot.scatter(title= 'Volatility comparison')

