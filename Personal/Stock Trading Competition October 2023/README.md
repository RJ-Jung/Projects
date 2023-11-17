# Introduction
If you're familiar with stock trading you may have heard of demo/simulated trading or proprietary trading firms as well as brokers or exchanges that often run competitions in trading a specific commodity or instrument or whatever they offer, With the higest percent gainers at end of the time period being the winner using whatever stragety they prefer wiout breaching the terms & conditions. These strategies could range from pure instinct and gambaling to proprietary coded algorithmic trading and everything in between. Im going to be analyzing the trades taken personally on one of these competitions as well as the first place contestant to find patterns and trends in successful trading history and improve future trading results. 
***
### Gathering the Data
Now most trading platforms allow you to export trade history as an CSV or PDF for tax purposes directly but since this was a temporary competition for just the month of October this option is unavalible and the login credentials no longer active. Luckily the platform does save the data of competitions ran throughout the year and it'll still viewable by changing the weblink address to the right location. This gives us access to the competition dashboard which looks like this.

<img src="https://github.com/RJ-Jung/Projects/blob/main/Personal/Stock%20Trading%20Competition%20October%202023/Resources/Images/octtradingpage.JPG?raw=true" width=50% height=50%>
<img src="https://github.com/RJ-Jung/Projects/blob/main/Personal/Stock%20Trading%20Competition%20October%202023/Resources/Images/octtradingpage2.JPG?raw=true" width=50% height=50%>

Initially I tried to automate gathering the data using webscraping methods but found some conflictions with the website formatting and split pages for the data table, I opted to copy paste the data from the table on the dashboard manually page by page into [text files](Personal/Stock Trading Competition October 2023/Resources/Data) as it was more convenient and faster than configuring a compatible web scraper for the site. Now I can upload these files into Excel for simple formatting adjustments as it currently looks like this!

<img src="https://github.com/RJ-Jung/Projects/blob/71db18d3d7a93f3ff6fa1c2e1bae43b4c937f495/Personal/Stock%20Trading%20Competition%20October%202023/Resources/Images/InitialExcelUpload.JPG?raw=true" width=80% height=80%>

I've uploaded the corrected format as a CSV accessible here:		[Personal Data](https://github.com/RJ-Jung/Projects/blob/ce83ca8e9c24a4dbfc88c894bc1e24c3d582709e/Personal/Stock%20Trading%20Competition%20October%202023/Resources/Data/OCTTradesPersonal.csv)		[Winner's Data](https://github.com/RJ-Jung/Projects/blob/main/Personal/Stock%20Trading%20Competition%20October%202023/Resources/Data/OctTradeFirstPlace.csv)

***
Im going to be using Microsoft SQL Server to analyze both data sets
before we start analysis I noticed the first place contestant had open trades on the final day of October which wasnt counted in the final result so we need to remove these trades from the dataset. 
Using the statement
```
DELETE FROM dbo.OctTradeFirstPlace    
WHERE close_time >= '2023-10-31 23:59:59';
```
10 Trades were removed as they were not counted in the competition and closed automatically after time time period ended.

I also realized the data set I downloaded nor the introduction went into detail about the rules and conditions of the competition, So to sum it up
  - The starting balance of the account is $100,000
  - You have a maximum static loss allowence of 10% of the starting value or equity value of $90,000
  - You also have a dynamic loss allowence of 5% or $5000 per day, the equity value or balance of the account at the end of the day will be the new floor for this rule (whichever is higher)
  - 5 Lot Max for Forex Pairs, 3 Lot Max for Indices, 3 lot for Crypto and Commodities.
  - No automated trading is allowed

Knowing this we should create a column for the account value per trade which wasn't included in the data we took. As for the strategies used we will assume unknown or proprietary for both.

I tried to do this in SQL with a statement like this using common table expression after creating a row_number column ordered by opening trade times
```
WITH cte AS (
    SELECT
        ROW_NUMBER() OVER (ORDER BY Open_Time) AS row_num,
        Open_Time,
        profit,
        CASE WHEN row_num = 1 THEN 100000
             ELSE COALESCE(LAG(account_value) OVER (ORDER BY Open_Time), 0) END AS account_value
    FROM dbo.OctTradeFirstPlace
)

UPDATE dbo.OctTradeFirstPlace
SET account_value = cte.account_value
FROM dbo.OctTradeFirstPlace
JOIN cte ON dbo.OctTradeFirstPlace.Open_Time = cte.Open_Time;

SELECT *
FROM dbo.OctTradeFirstPlace;
```
but I had trouble getting this or any other method in SQL to work properly, if anyone has some advice or a method please let me know!
Instead I went pack to Excel to create the new column with simple functions. with the first row as `=100000 + H2` H2 being the fist cell with a profit value. and subsequest cells in the account value column J simply having `=J2 + H3` , `=J3 + H4` , `=J4 + H5` and so on. Using the prevous account value while adding the current profit value and filling the rest of the column with Excels automatic recognition of the formula structure.
<img src="https://github.com/RJ-Jung/Projects/blob/b873daa28204bbfe58d46cae0b93b1932b7cd28c/Personal/Stock%20Trading%20Competition%20October%202023/Resources/Images/excelaccvalue.JPG" width=50% height=50%>

I've uploaded the corrected format as a CSV accessible here:		[Personal Data](https://github.com/RJ-Jung/Projects/blob/c46fec5aa7536c883c68e38e6754669692bb1f42/Personal/Stock%20Trading%20Competition%20October%202023/Resources/Data/OCTTradesPersonalACCVAL.csv)		[Winner's Data](https://github.com/RJ-Jung/Projects/blob/main/Personal/Stock%20Trading%20Competition%20October%202023/Resources/Data/OctTradeFirstPlaceACCVAL.csv)

We can now start analysis
<details>
  <summary>Click me</summary>

```-- Queries for dbo.OctTradeFirstPlaceACCVAL

SELECT COUNT(*) AS Row_Count
FROM dbo.OctTradeFirstPlaceACCVAL;

SELECT MAX(ACC_Val) AS Max_Account_Value
FROM dbo.OctTradeFirstPlaceACCVAL;

SELECT MIN(ACC_Val) AS Min_Account_Value
FROM dbo.OctTradeFirstPlaceACCVAL;

SELECT ACC_Val,
       LAST_VALUE(ACC_Val) OVER (ORDER BY Open_Time) AS Last_Account_Value
FROM dbo.OctTradeFirstPlaceACCVAL
ORDER BY Open_Time DESC
OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY;

SELECT AVG(profit) AS Average_Profit
FROM dbo.OctTradeFirstPlaceACCVAL
WHERE profit > 0;

SELECT AVG(profit) AS Average_Loss
FROM dbo.OctTradeFirstPlaceACCVAL
WHERE profit < 0;

SELECT AVG(profit) AS Combined_Profit_Average
FROM dbo.OctTradeFirstPlaceACCVAL;

SELECT MAX(profit) AS Max_Profit
FROM dbo.OctTradeFirstPlaceACCVAL;

SELECT MIN(profit) AS Max_Loss
FROM dbo.OctTradeFirstPlaceACCVAL;

-- Queries for dbo.OCTTradesPersonalACCVAL

SELECT COUNT(*) AS Row_Count
FROM dbo.OCTTradesPersonalACCVAL;

SELECT MAX(ACC_Val) AS Max_Personal_Account_Value
FROM dbo.OCTTradesPersonalACCVAL;

SELECT MIN(ACC_Val) AS Min_Personal_Account_Value
FROM dbo.OCTTradesPersonalACCVAL;

SELECT ACC_Val,
       LAST_VALUE(ACC_Val) OVER (ORDER BY Open_Time) AS Last_Personal_Account_Value
FROM dbo.OCTTradesPersonalACCVAL
ORDER BY Open_Time DESC
OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY;

SELECT AVG(profit) AS Average_Profit
FROM dbo.OCTTradesPersonalACCVAL
WHERE profit > 0;

SELECT AVG(profit) AS Average_Loss
FROM dbo.OCTTradesPersonalACCVAL
WHERE profit < 0;

SELECT AVG(profit) AS Combined_Profit_Average
FROM dbo.OCTTradesPersonalACCVAL;

SELECT MAX(profit) AS Max_Profit
FROM dbo.OCTTradesPersonalACCVAL;

SELECT MIN(profit) AS Max_Loss
FROM dbo.OCTTradesPersonalACCVAL;
```
  </details>

With the statements above we can see generate generate some basic datapoints
| Winner                  	|            	| Personal                	|           	|
|-------------------------	|------------	|-------------------------	|-----------	|
| Trades Taken            	| 112        	| Trades Taken            	| 78        	|
| Max Acc Value           	| 151,392.23 	| Max Acc Value           	| 105631    	|
| Min Acc Value           	| 101440.86  	| Min Acc Value           	| 99783     	|
| Last Acc Value          	| 151386.81  	| Last Acc Value          	| 105205.40 	|
| Average Profit          	| 811.49     	| Average Profit          	| 189.71    	|
| Average Loss            	| -256.08    	| Average Loss            	| -130.02   	|
| Combined Average Profit 	| 458.81     	| Combined Average Profit 	| 66.73     	|
| Max Profit              	| 6720       	| Max Profit              	| 1450.19   	|
| Max Loss                	| -2520.90   	| Max Loss                	| -507.44   	|
| Average Volume          	| 2.35       	| Average Volume          	| 0.5225    	|

Addionally I from counting the positive and negative values in the profit column we find that while I had a `63.24%` winrate first place had `66.96%`.
The winner had almost 5x the volume per trade, trading more agressively compared to my trade sizing.
The winner also never dipped below the starting amount of 100,000 gaining 1440.86 his first trade while I wasnt as lucky.
The winner also took 34 more trades than I did.

```
SELECT Symbol, COUNT(*) AS count
FROM dbo.OctTradeFirstPlace
WHERE Symbol IS NOT NULL
GROUP BY Symbol
ORDER BY count DESC;

SELECT Symbol, COUNT(*) AS count
FROM dbo.OCTTradesPersonal
WHERE Symbol IS NOT NULL
GROUP BY Symbol
ORDER BY count DESC;
```
From the query above we also see that while I stuck to trading just one symbol the winner traded multiple.
| Winner  	|       	| Personal 	|       	|
|---------	|-------	|----------	|-------	|
| Symbol  	| count 	| Symbol   	| count 	|
| US30x   	| 34    	| US30x    	| 78    	|
| NAS100x 	| 19    	|          	|       	|
| GER40x  	| 14    	|          	|       	|
| FRA40x  	| 8     	|          	|       	|
| UK100x  	| 7     	|          	|       	|
| XAUUSDx 	| 7     	|          	|       	|
| EURUSDx 	| 4     	|          	|       	|
| GBPJPYx 	| 3     	|          	|       	|
| CADJPYx 	| 3     	|          	|       	|
| EURJPYx 	| 2     	|          	|       	|
| XAGUSDx 	| 2     	|          	|       	|
| USDJPYx 	| 1     	|          	|       	|
| CHFJPYx 	| 1     	|          	|       	|
| EURAUDx 	| 1     	|          	|       	|
| AUDCADx 	| 1     	|          	|       	|
| AUDCHFx 	| 1     	|          	|       	|
| AUDUSDx 	| 1     	|          	|       	|
| GBPAUDx 	| 1     	|          	|       	|
| GBPCADx 	| 1     	|          	|       	|
| GBPCHFx 	| 1     	|          	|       	|

With Tableau we can create some quick graphics from the dataset such as a growth curve.

<img src="https://github.com/RJ-Jung/Projects/blob/main/Personal/Stock%20Trading%20Competition%20October%202023/Resources/Images/firstplaceaccvalgrowth.JPG" width=50% height=50%>

Here we can see the winners account value curve from days he traded.

***
### Conclusions

Considering my personal winrate of `63%` to the Winners `67%`, we know that the the chance of losing my first trade is `37%` losing twice in a row is `18.5%` then `9.25%` and so on, although there are multiple other varibles to consider when determing the outcome of a trade as the market changes everyday we can assume that if following a consistent entry and exit stragety. I could've definetly afforded to trade with a higher volume per day or total without risk of breaching the 10,000$ daily allowence or a 10% of the account value as a loss streak of 5 or more consecutively is well below 1% and highly unlikely.  In fact after the first day assuming we are in profit I could've even risked the entire 10,000$ daily allowence on a 63% chance and simply waitied until the next day for the allowence to reset to try again with an 18.5% chance of losing. This would still be profitable assuming we contue to gain more profit per successful trade than profit lost per unsuccessful trade. Which both the Winner as well as I were able to manage although with the winner having much more success with `811.49` per win and  `-256.08` per loss compared to my `189.71` per win and `-130.02` per loss.
