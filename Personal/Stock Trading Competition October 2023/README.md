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
