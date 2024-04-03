---
layout: post
title: Using SQL to run descriptive analytics on census data, part 2
categories: [p]
---

Continuing my analysis of this Los Angeles census dataset, I used SQL to run queries about home ownership and rent prices. In exploring these parts of the data, my aim was to deploy descriptive analytics to track how various rent- and ownership-related metrics changed over time.

**How many men own vs. rent? How many women? What is the average home value and average rent in this census district?**

<img width="603" alt="Screenshot 2024-04-03 at 1 36 05 PM" src="https://github.com/joshcode4/joshcode4.github.io/assets/160261781/f9b87fe9-d1bb-4692-b6c0-6e6227ae6130">

{% highlight c %}
select census_year,

count(distinct 
case when owned_or_rented_id=1 and sex_id=0 then resident_id end) homeowner_men,
count(distinct 
case when owned_or_rented_id=1 and sex_id=1 then resident_id end) homeowner_women,
count(distinct 
case when owned_or_rented_id!=1 and sex_id=0 then resident_id end) renter_men,
count(distinct 
case when owned_or_rented_id!=1 and sex_id=1 then resident_id end) renter_women,

concat('$', round((sum(case when owned_or_rented_id=1 then value_or_rent end)/count(distinct 
case when (owned_or_rented_id=1 and value_or_rent is NOT NULL) then resident_id end)),2)) avg_home_value,
concat('$', round((sum(case when owned_or_rented_id=0 then value_or_rent end)/count(distinct 
case when (owned_or_rented_id=0 and value_or_rent is NOT NULL) then resident_id end)),2)) avg_rent

from census_data_final c
group by census_year;
{% endhighlight %}

**Who in this census district had a rent increase between 1930 and 1940? How much did each person's rent increase? 
What was the average rent hike in this census district?**

<img width="519" alt="Screenshot 2024-04-03 at 1 37 42 PM" src="https://github.com/joshcode4/joshcode4.github.io/assets/160261781/c8f06262-2688-41e8-b822-10e5bfa20453">

{% highlight c %}
select c.resident_id, concat(r.first_name, ' ', r.last_name) as name, 
concat('$', c.value_or_rent) as 'rent_1930', concat('$', c2. value_or_rent) as 'rent_1940',
concat('$', (c2.value_or_rent-c.value_or_rent)) as rent_difference,
concat('$', round((q.rent_1940-q.rent_1930),2)) as avg_rent_difference
from census_data_final c
join resident_ids r on r.resident_id=c.resident_id
join census_data_final c2 on c.resident_id=c2.resident_id and c2.census_year=1940
join (select sum(c.value_or_rent)/count(distinct c.resident_id) as rent_1930, 
sum(c2.value_or_rent)/count(distinct c2.resident_id) as rent_1940
from census_data_final c
join census_data_final c2 on c.resident_id=c2.resident_id and c2.census_year=1940
where c.census_year=1930 and c.owned_or_rented_id=0 and c.value_or_rent is NOT NULL
and c2.value_or_rent is NOT NULL and c2.owned_or_rented_id=0) q
where c.owned_or_rented_id=0 and c.value_or_rent is NOT NULL
and c.census_year=1930 and c2.value_or_rent>c.value_or_rent;
{% endhighlight %}
