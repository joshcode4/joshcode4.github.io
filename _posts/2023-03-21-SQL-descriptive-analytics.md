---
layout: post
title: Using SQL to run descriptive analytics on census data
categories: [p]
---

While in college, I was hired by the Bunker Hill Refrain project to run analytics on census data in order to learn about the demographics of who lived in Bunker Hill, a Los Angeles neighborhood that was razed for freeway construction and urban renewal. 
The purpose of the Bunker Hill Refrain project was to synthesize oral histories with data analysis. Before applying my data science skills, I conducted multiple interviews with former residents and their children in order to get a qualitative picture of what life was like in this neighborhood. 

My next step was to conduct data analysis using SQL. I went to the city records office and scanned old census records, then I input them into excel, then from there into MySQL. I chose SQL to run descriptive analytics on the data because of its ability to query large amounts of data and aggregate in flexible ways. 

**What was the average income of employed residents in each census year?**

<img width="155" alt="Screenshot 2024-03-21 at 1 51 59 PM" src="https://github.com/joshcode4/joshcode4.github.io/assets/160261781/618e382e-736d-4ce5-a090-dc9723dbb687">

{% highlight c %}
select census_year,
concat('$', round((sum(c.income_received)/count(distinct c.resident_id)),2)) as avg_income
from census_data_final c
where work_id<=1
group by census_year;
{% endhighlight %}

**How did the number of employed men and employed women change from 1930-1940? What was the average wage gap between the sexes in each year?**

<img width="703" alt="Screenshot 2024-03-21 at 1 51 42 PM" src="https://github.com/joshcode4/joshcode4.github.io/assets/160261781/a2bd3dfe-5912-4eb3-be06-1d581b59d5bc">

{% highlight c %}
select census_year,

count(distinct 
case when work_id<=1 and sex_id=0 then resident_id end) employed_men,
count(distinct 
case when work_id<=1 and sex_id=1 then resident_id end) employed_women,
count(distinct
case when work_id=2 and sex_id=0 then resident_id end) unemployed_men, 
count(distinct
case when work_id=2 and sex_id=1 then resident_id end) unemployed_women, 

concat('$', round((sum(case when sex_id=0 then income_received end)/count(distinct 
case when work_id<=1 and sex_id=0 then resident_id end)),2)) avg_income_men,
concat('$', round((sum(case when sex_id=1 then income_received end)/count(distinct 
case when work_id<=1 and sex_id=1 then resident_id end)),2)) avg_income_women

from census_data_final c
group by census_year;
{% endhighlight %}

**Who changed occupations between 1930 and 1940?**
<img width="644" alt="Screenshot 2024-03-21 at 1 51 12 PM" src="https://github.com/joshcode4/joshcode4.github.io/assets/160261781/1c8d51e3-d459-45de-8698-9c2a29da526f">
{% highlight c %}
select c.resident_id, concat(r.first_name, ' ', r.last_name) as name, 
c.occupation as '1930_occupation', c.industry as '1930_industry',
c2.occupation as '1940_occipation', c2.industry as '1940_industry'
from census_data_final c
join resident_ids r on r.resident_id=c.resident_id
join census_data_final c2 on c.resident_id=c2.resident_id and c2.census_year=1940
where c.work_id<=1 and c.census_year=1930 and c.occupation!=c2.occupation;
{% endhighlight %}

**Who saw an increase in income between 1930 and 1940? What was the average increase in USD?**
<img width="1025" alt="Screenshot 2024-03-21 at 1 50 56 PM" src="https://github.com/joshcode4/joshcode4.github.io/assets/160261781/db8d3cc8-ddba-494e-acf0-16216c20a64e">
{% highlight c %}
select c.resident_id, concat(r.first_name, ' ', r.last_name) as name, 
c.occupation as '1930_occupation', c.industry as '1930_industry', 
c2.occupation as '1940_occipation', c2.industry as '1940_industry', 
concat('$', c.income_received) as '1930_income', concat('$', c2. income_received) as '1940_income',
concat('$', (c2.income_received-c.income_received)) as income_difference,
concat('$', round((q.income_1940-q.income_1930),2)) as avg_income_difference
from census_data_final c
join resident_ids r on r.resident_id=c.resident_id
join census_data_final c2 on c.resident_id=c2.resident_id and c2.census_year=1940
join (select sum(c.income_received)/count(distinct c.resident_id) as income_1930, 
sum(c2.income_received)/count(distinct c2.resident_id) as income_1940
from census_data_final c
join census_data_final c2 on c.resident_id=c2.resident_id and c2.census_year=1940
where c.census_year=1930 and c.work_id<=1 and c2.work_id<=1) q
where c.work_id<=1 and c.census_year=1930 and c2.income_received>c.income_received;
{% endhighlight %}
