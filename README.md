Bank Customer Behavior Monitoring – Case Study Project
Introduction
Financial institutions need to continuously monitor customer transactions for unusual patterns that might indicate fraud or money laundering. This project is a GitHub-ready data analytics case study demonstrating how to generate synthetic banking transaction data and analyze it for high-risk behaviors. We simulate bank transaction data (with fields like customer_id, amount, txn_time, merchant, txn_type, channel) and apply a set of rule-based checks to flag potentially suspicious transactions. The analysis is performed in a Jupyter Notebook (with Python libraries such as pandas, matplotlib, seaborn), and we discuss how the results could also be visualized in Power BI. Key features of this project:
Synthetic Data Generation: Use Python to create a realistic transaction dataset (customers, amounts, times, merchants, types, channels).
Exploratory Data Analysis (EDA): Visualize and summarize the transactions to understand typical behavior (e.g. distribution of amounts, transaction types).
Rule-Based Risk Identification: Implement business rules to flag high-risk transactions:
Amounts exceeding a threshold (e.g. $10,000)
flagright.com
More than a certain number of transactions per customer per day (e.g. >5)
austrac.gov.au
Transactions involving cash (CASH type)
amlwatcher.com
Non face-to-face transactions (e.g. online transactions)
sanctionscanner.com
Visualization of Results: Generate charts (using Python and optionally Power BI) to illustrate the flagged transactions and patterns.
Documentation & Reproducibility: Include a README explaining data schema, rules, and usage, plus a requirements file and .gitignore for easy GitHub publishing.
By the end of this case study, one can see how simple rules can help surface unusual customer activity for further investigation, and the project structure is suitable for sharing on GitHub or extending with real data.
Synthetic Data Generation
To facilitate testing and development without sensitive real data, we create synthetic banking transactions. The Python script (in the Jupyter Notebook) generates data with the following schema:
customer_id: Unique ID for a customer (e.g. C001, C002, ...).
amount: Transaction amount in USD.
txn_time: Timestamp of the transaction (date and time).
merchant: The merchant or transaction counterpart (examples: Amazon, ATM, Starbucks, UtilityBill).
txn_type: Type of transaction – e.g. PURCHASE, CASH (cash withdrawal/deposit), TRANSFER, PAYMENT, NON FACE TO FACE (a remote transaction).
channel: Channel through which the transaction was made – e.g. Online, Mobile, ATM, Branch.
We use random distributions to populate this data:
Transaction amounts follow an exponential distribution for everyday spending, but with a small probability of very large values to simulate high-value transactions (outliers).
Transaction times are spread over a period (e.g. one month) at random intervals.
Merchants are randomly chosen from a list of common merchants or transaction descriptions.
Transaction types and channels are assigned with realistic frequencies (for example, a majority might be PURCHASE and Online, with fewer CASH or Branch transactions).
To ensure the dataset includes scenarios for the high-frequency rule, the generator deliberately inserts a few days where a customer has more than 5 transactions in one day (e.g. one customer might have 6 transactions on the same day, another has 7). This way, we can test the rule that flags an unusually high number of daily transactions. Reproducibility: We set a random seed so that the synthetic data is reproducible. The generated dataset (saved as transactions_synthetic.csv in the data/ folder) contains on the order of 1,000+ transactions across ~50 customers.
Exploratory Data Analysis (EDA)
Before diving into risk rules, the project notebook performs EDA to understand the synthetic dataset's characteristics:
Peek at the Data: We display the first few rows of the dataset to ensure the format and values look plausible (e.g., correct columns, random but reasonable values).
Statistical Summary: Using pandas describe, we get summary statistics of the amount field (mean, median, min, max, quartiles) to see the range of transaction values.
Distribution of Amounts: We plot a histogram of transaction amounts. Typically, we observe many small transactions and a long tail of larger amounts. We mark the $10,000 threshold on this plot to see how many transactions exceed it (outliers beyond the red dashed line in the chart).
Transaction Counts by Type and Channel: We use bar charts to show how many transactions fall into each txn_type category (Purchase vs Cash vs Transfer, etc.) and each channel (Online, ATM, etc.). This gives a sense of customer behavior; for example, we might see that online purchases form the bulk of transactions while cash transactions are fewer.
sanctionscanner.com
amlwatcher.com
 (These EDA steps verify that the synthetic data behaves realistically and provide context for applying the risk rules.)
High-Risk Transaction Rules
We apply four simple rule-based criteria to tag transactions as "high-risk." These rules are commonly used as initial filters in fraud detection or compliance monitoring:
High Amount Transactions: Any transaction over a certain amount threshold is flagged. We use $10,000 as the threshold in this case study. Large cash transactions in excess of $10k often require regulatory reporting (Currency Transaction Report) in real banking
flagright.com
, so high-value transactions are generally considered risky or at least noteworthy.
High Frequency (Same Day): If a single customer performs more than 5 transactions in one day, those transactions are flagged. A rapid series of transactions in a short time can indicate structuring (breaking up a large amount into many small transactions) or other anomalous behavior
austrac.gov.au
. For example, normally a customer might make a few transactions per day; if they suddenly make, say, 10 transactions within hours, it could be an attempt to avoid detection or a sign of account takeover.
Cash Transactions: Any transaction of type CASH is flagged. Cash transactions are inherently higher risk because cash is untraceable and often used in illicit activities. Cash-intensive businesses and large cash dealings are known red flags in AML (Anti-Money Laundering) monitoring
amlwatcher.com
. Thus, even a modest cash deposit or withdrawal might be subject to scrutiny since it could combine with others to exceed thresholds or indicate suspicious patterns.
Non Face-to-Face Transactions: Any transaction labeled NON FACE TO FACE is flagged. These are transactions conducted remotely (online, phone, etc.) without in-person verification. Such transactions carry higher fraud risk because the bank cannot easily verify the customer's identity during the transaction
sanctionscanner.com
. For instance, an online purchase or transfer might be more likely associated with identity theft or hacking compared to an in-branch transaction where ID is checked.
In the Jupyter Notebook, we implement these rules step by step:
Compute the number of transactions each customer makes per day (using a group-by count in pandas).
Create new boolean columns (flags) for each rule:
flag_high_amount = True if amount > 10000.
flag_high_freq = True if transactions count for that customer-day > 5.
flag_cash = True if txn_type == 'CASH'.
flag_non_face = True if txn_type == 'NON FACE TO FACE'.
Combine these to a final flag_any_risk indicating if any of the above are True for a transaction.
Each flagged transaction is considered "high-risk" in the context of our scenario. The rules can catch different patterns: some transactions will be flagged because they are large in value, others because of how often the customer is transacting, and others purely due to the nature of the transaction (cash or remote). Note: These rules cast a wide net and will include false positives (legitimate transactions that just happen to be large or frequent). In practice, a bank's monitoring system might use more nuanced thresholds or additional rules. However, they serve as a straightforward example for this case study.
Results and Findings
After applying the rules, we get summary statistics on how many transactions were flagged by each criterion. For example, in one run with ~1000 synthetic transactions, we might find:
High Amount > $10k: 60 transactions flagged. (These represent the outliers above $10,000.)
High Frequency (>5/day): 13 transactions flagged. (This corresponds to two instances of customers having 6 and 7 transactions in a day, respectively, in our generated data.)
Cash Transactions: 88 transactions flagged. (All transactions classified as CASH type.)
Non Face-to-Face: 104 transactions flagged. (All transactions classified as NON FACE TO FACE type.)
Total High-Risk Transactions (any rule): 248 transactions flagged.
The total high-risk (248 in this example) is less than the sum of individual categories because some transactions trigger multiple flags. For instance, a single transaction could be a $20,000 cash withdrawal – this would count under both the high amount and cash categories, but it’s one transaction in the total count. We also examine a few examples of the flagged transactions by printing out sample rows. For instance, we might see a row like:
customer_id  amount    txn_time           merchant    txn_type        channel   flag_high_amount  flag_high_freq  flag_cash  flag_non_face
C037         15230.65  2025-01-04 04:40:44  AppleStore   PURCHASE        Online   True              False          False      False
C021         98.50     2025-01-15 09:10:00  ATM          CASH            Branch   False             True           True       False
C018         250.00    2025-01-20 12:05:20  OnlineShop   NON FACE TO FACE Online  False             True           False      True
...
In the above hypothetical snippet:
The first row shows a high-value purchase (flagged for amount >10k).
The second row shows a cash transaction that was part of more than 5 on that day (flagged for frequency and as cash).
The third row shows an online transaction that was part of a high-frequency day (flagged for frequency and non-face-to-face).
Such output helps illustrate how each rule contributes to flagging. Analysts can then further investigate these, knowing why each transaction was flagged.
Visualization of Flagged Transactions
To communicate the findings, we include several visualizations in the notebook (and in the project folder's images/):
High-Risk Transactions by Rule: A bar chart (see figure below) displays the number of transactions flagged by each rule category. This gives an at-a-glance view of which factors are most frequently triggering alerts in our dataset. For example, in our synthetic data, non-face-to-face transactions were the most common trigger (since we labeled many transactions as such), followed by cash transactions, then high-value transactions, and far fewer triggered the frequency rule. 

Bar chart: Number of transactions flagged under each risk rule (High Amount, High Frequency, Cash, Non Face-to-Face).
Distribution with Threshold: The histogram of transaction amounts mentioned earlier visually shows the $10k threshold. We can easily see that most transactions are below it, and only a small tail of the distribution exceeds $10k (those are flagged by the high amount rule).
Transactions per Day: (Optional) A timeline or bar chart could show the number of transactions each day per customer, highlighting days that exceeded 5 transactions. This could be a way to spot the high-frequency anomalies in a calendar view. In this project, we mostly rely on the summary count, but this could be added for completeness.
All Python-generated charts are saved and included. The visuals confirm the rule logic is working (e.g., the count of high-frequency flags corresponds to the inserted scenarios of 6+ transactions in a day).
Power BI Dashboard (Optional)
In a real-world setting, analysts often use tools like Power BI to interactively explore such data. While this project’s core analysis is in Python, the synthetic data can be easily imported into Power BI to create a dashboard. For example, a Power BI report could include visuals similar to our Python plots:
A card showing the total number of high-risk transactions identified.
A bar chart of count of high-risk transactions by rule (like the one above).
A table listing flagged transactions with filters for customer or date, enabling drill-down.
Perhaps a time series chart of daily transaction volume with markers when high-risk events occurred.
(We have not provided a .pbix file, but the README outlines how one could replicate the analysis in Power BI. Screenshots or additional Power BI steps are mentioned as optional in the documentation.)
Project Structure and Usage
The project repository is organized as follows:
README.md – Detailed description of the project, dataset, rules, and instructions (the document you are reading summarizing the case study).
Bank_Transactions_Analysis.ipynb – Jupyter Notebook containing the Python code for data generation, EDA, rule implementation, and visualizations. You can step through this notebook to reproduce the analysis.
data/transactions_synthetic.csv – The synthetic dataset (CSV file) generated by the notebook. This is included for convenience so that running the notebook is optional to get the data (you could load this file directly in Power BI or other tools).
images/ – Sample image outputs (graphs) used for documentation. For example, risk_rule_flags.png is the bar chart of flagged transaction counts by rule.
requirements.txt – List of Python packages used (pandas, numpy, matplotlib, seaborn, etc.) to ensure the environment can be set up consistently.
.gitignore – Git ignore file to exclude environment files, caches (e.g., __pycache__, Jupyter checkpoints, etc.) from the repository.
How to run this project:
Install Requirements: Ensure you have Python 3 and Jupyter installed. Install the required libraries with pip install -r requirements.txt.
Open the Notebook: Launch Jupyter Notebook/Lab and open Bank_Transactions_Analysis.ipynb.
Run the Analysis: Execute the notebook cells in order. The notebook will generate the synthetic data (or you can skip generation and load the CSV), perform EDA, apply risk flagging rules, and produce output tables and charts.
Examine Results: Observe the printed outputs (counts of flagged transactions, sample flagged transactions) and the charts. Feel free to adjust the thresholds (Y = 10000, X = 5) or the data generation logic to see how the results change.
Power BI (optional): Load the CSV data into Power BI if you wish to create an interactive dashboard. You can recreate the visuals from the notebook and use slicers/filters for exploration. For example, filter by a specific customer to see only their transactions and which ones would be flagged.
The entire project is self-contained. By downloading the repository folder and running the notebook, you can reproduce the scenario. This case study can serve as a template for building more complex transaction monitoring systems or as a portfolio project showcasing data analytics skills in Python and BI tools.
Download the Project Folder
You can download the complete project folder as a zip file and upload it to GitHub or open it locally: Download: BankCustomerBehaviorMonitoringCaseStudy.zip Unzip this file to obtain the full project folder with all the files described. You can then open the notebook or inspect the README and code. (The repository is ready to be uploaded to GitHub as-is, with all necessary files and documentation.)
