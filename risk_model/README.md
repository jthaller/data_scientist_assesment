## **Clair Data Science Assessment**

### **Background**

At Clair, we’re on a mission to create financial freedom for America's workers by giving them access to free wage advances, allowing them to get paid as soon as they clock out of work. To reduce losses and ensure we provide wage advances responsibly, we need a data-driven method to determine which users are eligible for advances—and ideally, how much we can safely offer.

### **Objective**

Your task is to build a model that predicts whether a user is likely to default on a wage advance. Based on this, you’ll define an eligibility rule that determines whether we should **accept or reject** a user's advance request.

As a **bonus**, you may also propose a method for estimating the **maximum safe advance amount** per user/request to help further reduce potential losses.

This assessment is intended to take approximately 3–4 hours. We are not expecting a perfect solution — our goal is to evaluate your **approach to modeling, feature engineering, sampling, and business-oriented decision-making**. A model ROC AUC of 0.65 is completely acceptable.

### **Dataset**

Use the `loan_characteristics` table available in our cloud-hosted PostgreSQL database:

* **Host**: `pg-285c4482-getclair-7a3a.a.aivencloud.com`
* **Port**: `10594`
* **Database**: `assessment`
* **Username and password**: (will be shared via email)

### **Table Columns**

* `USER_ID`: Unique identifier of the user
* `BUSINESS_ID`: Employer ID
* `STARTED_ONBOARDING_AT`: When the user registered at Clair
* `HIRE_DATE`: When the user was hired by their current employer
* `LOAN_CREATION_DT`: When the wage advance was requested
* `DUE_DATE`: When the loan was due
* `DATE_OF_LAST_PAYCHECK`: Date of the user’s most recent paycheck
* `LOAN_AMOUNT`: Total amount of taken advance 
* `UNIQUE_PAY_PERIODS`: Number of distinct pay cycles observed advancing with us
* `ON_TIME_PAYMENTS`: Count of on-time repayments
* `LATE_PAYMENTS`: Count of late repayments
* `TOTAL_BANK_ACCOUNTS`: Total number of bank accounts linked
* `TOTAL_OPEN_ACCOUNTS`: Number of currently open accounts
* `TOTAL_CLOSED_ACCOUNTS`: Number of closed accounts
* `TOTAL_CURRENT_BALANCE`: Total balance across all linked accounts
* `TOTAL_AVAILABLE_BALANCE`: Available balance across accounts (e.g., in checking)
* `ALL_PAID_BY_CHECK_PAYCHECKS`: Indicator of paycheck delivery method (e.g., check vs direct deposit)
* `PAYCHECK_5AGO` to `PAYCHECK_1AGO`: Historical paycheck amounts for the last five cycles
* `INDUSTRY_LABEL`: Industry of the user’s employer
* `DEFAULT_FLAG`: Target variable — `1` if the user defaulted, `0` otherwise

---

### **Part I: Risk Model**

Build a classification model to predict `DEFAULT_FLAG`. 

---

### **Part II: Business Proposal**

Write a brief document that could be shared with **non-technical stakeholders** to explain:

* How the model works
* How the model supports lending decisions
* The expected business benefits (e.g., loss reduction, improved user targeting)
* (Bonus) How can your method estimate a safe loan amount per user

---

### **Deliverables**

Please submit the following:

* 📓 A **Jupyter Notebook** with your full modeling process, code, and commentary (we do not expect a production-ready model from you for this assessment; however, it is required for your work) 
* 📄 A short **stakeholder-facing document** summarizing the model and its business value

Let us know if you have any issues accessing the database or need clarification on the task. Good luck—we’re excited to see your approach!

