# QuadrantSignal
---

##### View this presentation with Microsoft Sway:
https://sway.office.com/R0z9xbZqCletIpU6?ref=Link

---

![Cover Image](Images/CoverImage.jpg)


Building on our revenue analysis in [ValuationVisualization](https://github.com/marshallwolfe/ValuationVisualization) we created an email signal
generator and deployed it using AWS. Additionally, we conducted a deeper analysis of
revenue growth and revenue acceleration on a quarterly basis to increase our sample size,
signal frequency, and see if we could further validate our initial quadrant findings.

![Quadrant Compass](Images/Directional_Quadrant_Graphic.png)

## Supporting Analysis

* For a more complete set of data and to increase the number of data points/frequency of signals we pulled data from www.stockrow.com.

* The quarterly data of the Dow 30 stocks (excluding Dow Inc. (DOW) due to limited data from the spin off of “Dow” from DowDuPont) was transformed using Excel Power Query and saved as csvs to import prepared DataFrames for analysis and machine learning.

* Calculating the mean Following Quarter Return we found that the upper right quadrant (“Strong Buy” signal) does indeed outperform the other three quadrants with approximately double the return of the next closest quadrant. The frequency and lookback period for this calculation was quarterly returns over the last 10 years.

![Quadrant Return Bar Chart](Images/BarChart1.png)

* To take a deeper look, we calculated the excess returns above the index (DJI) to analyze market-neutral performance of each quadrant. The resulting bar plot distinctly confirms our quadrant-based trade signal generation:
    * Upper Right (Strong Buy) has the highest excess return over the 10 years of quarterly studied
    * Upper Left (Sell) has the lowest excess returns
    * Bottom Right (Hold) has the next lowest excess returns
    * Bottom Left (Consider Buying) has the second highest excess returns

![Quadrant Excess Return Bar Chart](Images/BarChart2.png)

## QuadrantSignal Email Generator
* We increased the signal frequency from annual to quarterly by modifying our original code to call the quarterly fundamental data from FinancialModelingPrep.com’s API.

![Financial Modeling Prep API Quarterly Financials](Images/FinancialModelingPrepAPIQuarterlyData.png)

* To automate a daily list of recently announced earnings tickers we called data from a Yahoo Earnings Calendar Scraper API.
    * Since earnings are announced before, during and after the bell we set up our python script to run once daily before the bell pulling tickers for earnings announced the prior day.
    * An additional reason for this is to allow time for the Financial Modeling Prep API to update with the latest earnings announcement data.
    * A cutoff date of 90 days was put in place to remove tickers that did not have updated api data for the most recent quarter announcement and avoid reporting a QuadrantSignal for a previous quarter.

![Yahoo Earnings Calendar API](Images/yahoo-earnings-calendar.jpg)

* Only the previous three quarters of fundamental data is pulled to speed processing and meet the minimum requirements for calculating quarter-over-quarter revenue acceleration.
* We defined a “quadrant_assignment” function into which we passed our DataFrame. The function initialized two new columns called “Quadrant” and “Signal.” Revenue Growth is the horizontal axis of our compass, while Revenue Acceleration is our vertical axis.
    * We assigned tickers with accelerating (+) revenue growth (+) into the “Upper Right” quadrant of our compass with a signal of “Strong Buy.”
    * We assigned tickers with decelerating (-) revenue growth (+) into the “Lower Right” quadrant with a “Hold” Signal.
    * We assigned tickers with decelerating (-) declining revenue (-) into the “Lower Left” quadrant with a Signal of “Potential Buy” because our analysis shows that some companies in this quadrant often have quite positive returns over the following period due to turnaround scenarios.
    * We assigned tickers with accelerating (+) revenue decline (-) into the “Upper Left” quadrant with a Signal of “Sell.”

![AWS Cloudcraft Graphic](Images/ServerlessApplicationArchitectureQuadrantSignal.png)

* We turned our Jupyter Notebook code into a modular format. An executable Python script was exported and other required libraries (request, pandas, and yahoo-earnings-calendar) were packaged with Docker into a zip file. The zip file was then uploaded to AWS Lamdba.
* This included the creation of two functions for message handling. We defined a “create_message” function into which we passed our DataFrame. This function generates the body of our daily emails. An introductory line is followed by a table that is created by a big string to display “Ticker: Signal” at the top followed by a line separating the header of the table from the body of the table which consists of each ticker that announced earnings yesterday and that ticker’s Signal. The footer contains a branded logo footerwith legal disclaimer text embedded in the image.
* To integrate our logo in a branded email footer with a legal disclaimer, we used AWS S3 to host an image which is hyperlinked in the QuadrantSignal email body.
* Automating the run process of our code we utilized AWS CloudWatch to create an Event Rule that triggers the python script to run each day at 7am CDT (12:00pm UTC).
* The AWS Lambda function runs our QuadrantSignal executable code which generates our email body text and publishes the email via AWS SNS. AWS SNS also hosts our list of subscribers.
* The final product sends an email with QuadrantSignals before the Market Open Monday through Friday (and an email Saturday and Sunday with the occasional earnings announcements that can be acted upon on Monday’s open). The QuadrantSignal email simply provides a list of tickers that reported earnings the previous day with our QuadrantSignal that is based on our previously researched revenue analysis.

![Email Example](Images/Email_Mockup.jpg)

## Looking Forward
* Create a subscription model service
* Add inline image rendering for our branded emails
* Increase email frequency by adding intraday and after market alerts
* Track live performance by quadrant
* Add predictive analytics with machine learning