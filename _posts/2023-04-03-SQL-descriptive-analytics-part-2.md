[rent_vs_own_by_year.csv](https://github.com/joshcode4/joshcode4.github.io/files/14856233/rent_vs_own_by_year.csv)---
layout: post
title: Using SQL to run descriptive analytics on census data, part 2
categories: [p]
---

Continuing my analysis of this Los Angeles census dataset, I used SQL to run queries about home ownership and rent prices. In exploring these parts of the data, my aim was to deploy descriptive analytics to track how various rent- and ownership-related metrics changed over time.

**How many men own vs. rent? How many women? What is the average home value and average rent in this census district?**

[Ucensus_year,homeowner_men,homeowner_women,renter_men,renter_women,avg_home_value,avg_rent
1930,2,3,65,58,$860,$10.41
1940,2,2,58,67,$1075,$12.82
ploading rent_vs_own_by_year.csv…]()

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

