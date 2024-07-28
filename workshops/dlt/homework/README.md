# Open source data ingestion for RAGs with dlt

# Homework

You can check the code in this [notebook](./homework_llm_RAG_dlt.ipynbhome)

In the workshop, we extracted contents from two pages in notion titled "Workshop: Benefits and Perks" and "Workshop: Working hours, PTO, and Vacation". 

Repeat the same process for a third page titled "Homework: Employee handbook" (hidden from public view, but accessible via API key):

1. Modify the REST API source to extract only this page.
2. Write the output into a separate table called "homework".
3. Remember to update the table name in all cells where you connect to a lancedb table.

To do this you can use the [workshop Colab](https://colab.research.google.com/drive/1nNOybHdWQiwUUuJFZu__xvJxL_ADU3xl?usp=sharing) as a basis.

Now, answer the following questions:  

## Q1. Rows in LanceDB

How many rows does the lancedb table "notion_pages__homework" have?

* 14
* 15
* 16
* 17

**Answer**: 14

The table `notion_pages___employee_handbook` has 14 rows.


## Q2. Running the Pipeline: Last edited time

In the demo, we created an incremental dlt resource `rest_api_notion_incremental` that keeps track of `last_edited_time`. What value does it store after you've run your pipeline once? (Hint: you will be able to get this value by performing some aggregation function on the column `last_edited_time` of the table)

* `Timestamp('2024-07-05 22:34:00+0000', tz='UTC') (OR "2024-07-05T22:34:00.000Z")`
* `Timestamp('2024-07-05 23:33:00+0000', tz='UTC') (OR "2024-07-05T23:33:00.000Z")`
* `Timestamp('2024-07-05 23:52:00+0000', tz='UTC') (OR "2024-07-05T23:52:00.000Z")`
* `Timestamp('2024-07-05 22:56:00+0000', tz='UTC') (OR "2024-07-05T22:56:00.000Z")`

**Answer**: * Timestamp('2024-07-05 22:34:00+0000', tz='UTC') (OR "2024-07-05T22:34:00.000Z")`
```text
last_edited_time
2024-07-18 14:00:00+00:00    1
2024-06-26 09:03:00+00:00    1
2024-06-26 09:08:00+00:00    1
2024-06-26 09:09:00+00:00    1
2024-06-26 09:11:00+00:00    1
2024-06-26 09:17:00+00:00    1
2024-06-26 09:20:00+00:00    1
2024-06-26 09:21:00+00:00    1
2024-07-03 17:34:00+00:00    1
2024-07-18 17:28:00+00:00    1
2024-07-03 17:26:00+00:00    1
2024-06-26 08:52:00+00:00    1
2024-07-03 17:19:00+00:00    1
2024-07-05 22:32:00+00:00    1
Name: count, dtype: int64
```

## Q3. Ask the Assistant 

Find out with the help of the AI assistant: how many PTO days are the employees entitled to in a year?  

* 20
* 25
* 30
* 35

**Answer**: 20

When we run the RAG system we get the result:
```python
You: how many PTO days are the employees entitled to in a year?
Assistant: The employees are entitled to [20] days of Paid Time Off (PTO) per year. You can take your PTO at any time after your first week with us and earn one additional day every year, up to a maximum of 25 days overall. If you want to use PTO, send a request through our HRIS system, or contact your manager or HR for approval. You do not have to specify a reason for taking PTO. However, if you leave our company, we may compensate accrued PTO with your final paycheck according to local law. If the law doesn't have provisions, we will compensate accrued leave to employees who were not terminated for cause.
If you are an exempt employee and worked on a holiday that was not in your calendar, you may receive an additional day of PTO within 12 months after that holiday. We count hours worked on holidays towards determining whether you're entitled to overtime pay if you work more than 40 hours per week.
You: 
```