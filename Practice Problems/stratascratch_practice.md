# StrataScratch SQL Practice Notes

My notes from practicing on StrataScratch, including problems, solutions, common mistakes, and key takeaways. This helps with interview prep and reinforcing SQL concepts.

## Table of Contents
- [Most Lucrative Products](#most-lucrative-products)
- [Compare Employee Salaries to Department Averages](#compare-employee-salaries-to-department-averages)
- [Customer Details with Optional Orders](#customer-details-with-optional-orders)
- [Salary Difference Between Departments](#salary-difference-between-departments)
- [Last Bike Usage Times](#last-bike-usage-times)
- [Formatting Queries Best Practices](#formatting-queries-best-practices)
- [Posts Reacted to with a Heart](#posts-reacted-to-with-a-heart)
- [Processed Rate of Tickets](#processed-rate-of-tickets)
- [Election Winner with Fractional Votes](#election-winner-with-fractional-votes)
- [General Takeaways](#general-takeaways)

---

## Most Lucrative Products

### Problem
You have been asked to find the 5 most lucrative products in terms of total revenue for the first half of 2022 (from January to June inclusive). Output their IDs and the total revenue.

### Initial Incorrect Solution
```sql
select product_id, (sum(units_sold) * cost_in_dollars) as revenue from online_orders
where date_sold between '2022-01-01' and '2022-06-30'
group by product_id
order by revenue desc
limit 5
```

### Correct Solution
```sql
select product_id, sum(units_sold * cost_in_dollars) as revenue from online_orders
where date_sold between '2022-01-01' and '2022-06-30'
group by product_id
order by revenue desc
limit 5
```

### Notes
- **Why It’s Wrong**: The aggregation (sum) is applied only to units_sold, but not to the multiplication with cost_in_dollars. This assumes a uniform cost per product across all orders, which is unlikely given real-world data where prices might change.
- **How to Avoid This in the Future**:
  - Check Aggregation Scope: When using sum, avg, etc., ensure the expression inside accounts for all variables that vary per row (e.g., cost_in_dollars here). If you aggregate one column (e.g., sum(units_sold)), you can’t multiply it by a non-aggregated column unless it’s constant.
  - Test with Sample Data: Run the query with LIMIT 5 on raw data to verify units_sold and cost_in_dollars values, then compare the aggregated result to ensure it matches your expectation.
  - Use Subqueries or CTEs: If unsure, break it down (e.g., calculate revenue per order in a subquery, then sum it), which can help catch errors.

---

## Compare Employee Salaries to Department Averages

### Problem
Compare each employee's salary with the average salary of the corresponding department. Output the department, first name, and salary of employees along with the average salary of that department.

### Solution
```sql
with avg_salary_cte as 
(select avg(salary) as dept_avg, department from employee
group by department)

select e.department, e.first_name, e.salary, cte.dept_avg
from employee e
join avg_salary_cte cte
on e.department = cte.department
```

### Notes
- This is a good use case for a CTE.
- When you need to look at the same data from a different angle and compare, you can use a CTE to aggregate and group it separately.

---

## Customer Details with Optional Orders

### Problem
Find the details of each customer regardless of whether the customer made an order. Output the customer's first name, last name, and the city along with the order details. Sort records based on the customer's first name and the order details in ascending order.

### Solution
```sql
select c.first_name, c.last_name, c.city, o.order_details from customers c
left join orders o
on o.cust_id = c.id
order by c.first_name, o.order_details
```

### Notes
- Pay attention to phrasing. Break down each component of the sentence and what it means for the query.
- “Regardless of whether there is an order” would lead me to consider the join type. A left join was needed to include all information requested.

---

## Salary Difference Between Departments

### Problem
Calculates the difference between the highest salaries in the marketing and engineering departments. Output just the absolute difference in salaries.

### My Solution
```sql
with cte as
(select
MAX(CASE WHEN d.department = 'marketing' THEN e.salary END) as marketing_max,
MAX(CASE WHEN d.department = 'engineering' THEN e.salary END)  as engineering_max
from db_employee e
join
db_dept d
on e.department_id = d.id)
select abs(marketing_max - engineering_max) from cte
```

### Official Solution
```sql
SELECT ABS(MAX(CASE  
                   WHEN dept.department = 'marketing' THEN emp.salary  
               END) - MAX(CASE  
                              WHEN dept.department = 'engineering' THEN emp.salary  
                          END)) AS salary_difference  
FROM db_employee emp  
JOIN db_dept dept ON emp.department_id = dept.id  
WHERE dept.department IN ('marketing',  
                          'engineering');
```

### Notes
- Multiple cases are nested not only within the query, but within a single math function.
- You don’t need extra queries for this. Keep it as concise as the syntax will allow.

---

## Last Bike Usage Times

### Problem
Find the last time each bike was in use. Output both the bike number and the date-timestamp of the bike's last use (i.e., the date-time the bike was returned). Order the results by bikes that were most recently used.

### Correct Solution
```sql
select bike_number, max(end_time) as recent from dc_bikeshare_q1_2012
group by bike_number
order by recent desc
```

### Takeaway
When you need the latest or earliest record per group (bike), use aggregation (MAX or MIN) with GROUP BY to get the correct value. Practice recognizing when a problem requires aggregating data (max, min, sum) versus filtering or deduplicating.

---

## Formatting Queries Best Practices

### Original Query
```sql
select id, first_name, last_name, department_id, max(salary) as current_salary 
from ms_employee_salary
group by id, first_name, last_name, department_id
order by id
```

### Using Best Practices
```sql
SELECT id,
       first_name,
       last_name,
       department_id,
       MAX(salary) AS salary
FROM ms_employee_salary
GROUP BY id,
         first_name,
         last_name,
         department_id
ORDER BY id ASC;
```

### Note
This is important because queries can be large and need clear formatting to be readable.

---

## Posts Reacted to with a Heart

### Problem
Find all posts which were reacted to with a heart. For such posts output all columns from facebook_posts table.

### Solution
```sql
SELECT DISTINCT p.post_date, 
    p.post_id, 
    p.post_keywords, 
    p.post_text, 
    p.poster 
FROM facebook_reactions r
JOIN facebook_posts p
ON r.post_id = p.post_id
WHERE r.reaction LIKE 'heart'
```

### Note
Always ask yourself if the DISTINCT keyword is needed.

---

## Processed Rate of Tickets

### Problem
Find the processed rate of tickets for each type. The processed rate is defined as the number of processed tickets divided by the total number of tickets for that type. Round this result to two decimal places.

### Solution
```sql
with CTE as
(select complaint_id, processed as was_processed
from facebook_complaints
where processed = true)

select fc.type, round(count(c.was_processed) / count(fc.processed), 2) as rate
from facebook_complaints fc
join CTE c
on fc.complaint_id = c.complaint_id
group by type
```

### Note
- Two ways of using a CTE: SELECT from a CTE for direct use of its results; JOIN a CTE to merge its data with other tables. Here, I needed to join.
- When you select from a CTE, you are basically doing the same as writing a subquery. But it is more readable.
- CTEs are also optimized for scalability. A CTE is computed once and can be referenced multiple times within the query. A subquery may be computed each time it is referenced.

---

## Election Winner with Fractional Votes

### Problem
The election is conducted in a city and everyone can vote for one or more candidates, or choose not to vote at all. Each person has 1 vote so if they vote for multiple candidates, their vote gets equally split across these candidates. For example, if a person votes for 2 candidates, these candidates receive an equivalent of 0.5 vote each. Some voters have chosen not to vote, which explains the blank entries in the dataset.
Find out who got the most votes and won the election. Output the name of the candidate or multiple names in case of a tie.
To avoid issues with a floating-point error you can round the number of votes received by a candidate to 3 decimal places.

### Solution
```sql
with CTE as 
(select voter, count(voter) as times_voted from voting_results
group by voter),

cte_2 as
(select vr.voter, 1.0 / c.times_voted as vote_value, candidate from voting_results vr
join CTE c
on c.voter = vr.voter),

cte_3 as
(select candidate, sum(vote_value) as total_votes from cte_2
where candidate is not null
group by candidate
order by total_votes desc)

select candidate from cte_3
group by candidate
having sum(total_votes) = (select max(total_votes) from cte_3)
```

### Note
- This was tough. But just talk it out in plain English. Take those words and group them into separate calculations.
- Then do one calculation at a time. You can just keep adding CTEs for each needed step in this particular case.

---

## General Takeaways
- Aggregation pitfalls: Always ensure operations like sum apply to the full expression (e.g., sum(a * b) vs. sum(a) * b).
- CTEs vs. Subqueries: Use CTEs for readability and when reusing calculations.
- Joins: Choose type based on requirements (e.g., LEFT for "regardless of").
- Query Formatting: Uppercase keywords, line breaks for clauses, aliasing for clarity.
- Testing: Use sample data, LIMIT, and breakdowns to verify.
- Advanced: Handle ties with HAVING, fractional values with decimals, and conditionals with CASE.