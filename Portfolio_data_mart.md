Pipeline Design - 
<img width="1264" height="372" alt="Screenshot 2026-07-05 at 7 45 51 AM" src="https://github.com/user-attachments/assets/33c0a5b9-759f-4bc9-9aa9-1a894092843e" />

Star Schema Design - 
<img width="645" height="686" alt="Screenshot 2026-07-05 at 7 46 59 AM" src="https://github.com/user-attachments/assets/fb0f6f59-d4fe-48a0-8fb6-af72613aa2ca" />

I would design a Portfolio Monitoring Mart in the Gold layer using a star schema. The central fact table would contain one record per loan or credit application, enriched with customer, bureau, fraud, AML, and repayment information. Around it, I would maintain dimensions such as Customer, Date, and Risk Band. This mart would power BI dashboards for the Risk team, enabling them to monitor KPIs like approval rate, portfolio exposure, delinquency (30/60/90+ DPD), fraud trends, AML alerts, and risk distribution without requiring complex joins across operational tables. The mart is optimized for analytical queries and refreshed on a scheduled basis (e.g., hourly or daily depending on business requirements).

Fact Table:
fact_portfolio_monitoring -One row per loan/application.

Example columns:
1. application_id
2.customer_id
3.application_date
4.decision (Approved / Rejected)
5.loan_amount
6.outstanding_balance
7.credit_score
8.fraud_score
8.aml_status
9.risk_band
10.loan_status (Active / Closed / Defaulted)
11.days_past_due (DPD)

Dimension Tables:
dim_customer:
1. customer_id
2. age_group
3. nationality
4. customer_segment

dim_date:
1. day
month
3. quarter
4. year

dim_risk:
1. risk_band
2. credit_score_bucket

Dashboard Metrics
The Risk team typically wants to monitor KPIs such as:

Portfolio Overview
1. Total applications
Total approved loans
3. Total rejected loans
4. Approval rate
5. Total loan exposure
6. Outstanding balance

Credit Risk
1. Average credit score
2. Loans by risk band
3. Distribution of credit scores
4. High-risk customers

Fraud Monitoring
1. Average fraud score
2. Fraud alerts
3. Fraud cases by month

Delinquency Monitoring
1. Loans past due
2. AML matches
3. Pending AML reviews
4. High-risk customers

