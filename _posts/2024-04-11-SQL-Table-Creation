---
layout: post
title: Creating Tables in MySQL
categories: [p]
---

In order to organize and run analytics on this Los Angeles Census Data, I created a parent table and several child tables in SQL. I then used the table import wizard to import the data from Excel into MySQL. 

{% highlight c %}
DROP TABLE if exists census_data_py;

CREATE TABLE census_data_py (
    resident_id int NOT NULL,
    street_id int,
    house_number int,
    age int,
    marital_id int,
    owned_or_rented_id int,
    school_id int,
    seeking_work_id int,
    occupation varchar(255),
    industry varchar(255),
    weeks_worked int,
    income_received int,
    work_id int,
    unemployed_reason_id int,
    value_or_rent int,
    PRIMARY KEY (resident_key),
    FOREIGN KEY (street_id) REFERENCES street_key(street_id),
    FOREIGN KEY (marital_id) REFERENCES marital_key(marital_id),
    FOREIGN KEY (owned_or_rented_id) REFERENCES owned_rented_key(owned_or_rented_id),
    FOREIGN KEY (school_id) REFERENCES school_key(school_id),
    FOREIGN KEY (seeking_work_id) REFERENCES seeking_work_key(seeking_work_id),
    FOREIGN KEY (work_id) REFERENCES work_key(work_id)
);

CREATE TABLE resident_key (
	last_name varchar(255),
    first_name varchar(255),
    sex varchar(255),
    race varchar(255),
    resident_id int NOT NULL
);

CREATE TABLE street_key (
	street varchar(255),
    street_id int
);

CREATE TABLE sex_key (
	sex char(1),
    sex_id int
);

CREATE TABLE marital_key (
	marital_status varchar(255),
    marital_description varchar(255),
    marital_id int
);

CREATE TABLE owned_or_rented_key (
	owned_or_rented char(1),
    owned_or_rented_description varchar(255),
    owned_or_rented_id int
);

CREATE TABLE school_key (
	in_school char(1),
    school_id int
);

CREATE TABLE seeking_work_key (
	seeking_work char(1),
    seeking_work_id int
);

CREATE TABLE work_key (
	private_work char(1),
    public_work char(1), 
    work_type char(1),
    work_description varchar(255),
    work_id int
);
{% endhighlight %}
