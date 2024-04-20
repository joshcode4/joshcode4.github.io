---
layout: post
title: Cleaning a dataset using Python
categories: [p]
---

Because the raw census data in my research project was scanned from old paper forms, the resultant dataset was pretty messy. I used Python to clean the data, converting two-response columns to binary integers and standardizing the values in each string column. I also used MySQL Connector to port in the key tables I created in Python into MySQL. Finally, I joined the primary and foreign keys to the main dataset using Python commands, thus creating a new dataset containing only integer values.

{% highlight c %}
import mysql.connector
from mysql.connector import Error
import pandas as pd
import numpy as np
from sqlalchemy import create_engine

x = pd.read_csv('census_data.csv')
x.head()
x.info()

# CLEANING DATA

x = x.dropna(how='all')

x.relationship_to_household.unique()

x.loc[(x['relationship_to_household'].str.contains('w', case=False, na=False)),'relationship_to_household']='Wife'
x.loc[(x['relationship_to_household'].str.contains('son', case=False, na=False)),'relationship_to_household']='Son'
x.loc[(x['relationship_to_household'].str.contains('dau', case=False, na=False)),'relationship_to_household']='Daughter'
x.loc[(x['relationship_to_household'].str.contains('hea', case=False, na=False)),'relationship_to_household']='Head'
x.loc[(x['relationship_to_household'].str.contains('lod', case=False, na=False)),'relationship_to_household']='Lodger'
x.loc[(x['relationship_to_household'].str.contains('inf', case=False, na=False)),'relationship_to_household']='Daughter'
x.loc[(x['relationship_to_household'].str.contains('wif', case=False, na=False)), 'relationship_to_household']='Wife'
x.loc[(x['relationship_to_household'].isnull()), 'relationship_to_household']='Unknown'
x['relationship_to_household'].unique()

x.marital_status.unique()
x.loc[(x['marital_status'].str.contains('w', case=False, na=False)), 'marital_status']='W'
x.loc[(x['marital_status'].str.contains('s', case=False, na=False)), 'marital_status']='S'
x.loc[(x['marital_status'].str.contains('M', case=False, na=False)), 'marital_status']='M'
x.loc[(x['marital_status'].str.contains('moved out', case=False, na=False)), 'marital_status']='D'
x.loc[(x['marital_status'].isnull()), 'marital_status']='U'

x.sex.unique()

x.loc[(x['sex'].str.contains('M', case=False, na=False)), 'sex']=0
x.loc[(x['sex'].str.contains('F', case=False, na=False)), 'sex']=1

x.loc[(x['private_work'].str.contains('Y', case=False, na=False)), 'private_work']=1
x.loc[(x['private_work'].str.contains('N', case=False, na=False)) | (x['private_work'].str.contains('-', case=False, na=False)) | (x['private_work'].isnull()), 'private_work']=0

x.loc[(x['public_work'].str.contains('Y', case=False, na=False)), 'public_work']=1

x.loc[(x['public_work'].str.contains('N', case=False, na=False)) | (x['public_work'].str.contains('-', case=False, na=False)) | (x['public_work'].isnull()), 'public_work']=0

x.loc[(x['owned_or_rented'].isnull()), 'owned_or_rented']='R'
x.loc[(x['owned_or_rented'].str.contains('0')),'owned_or_rented']='O'
x.owned_or_rented.unique()

x.loc[(x['in_school'].str.contains('N', case=False, na=False)) | (x['in_school'].isnull()), 'in_school']='N'
x.loc[(x['in_school'].str.contains('Y', case=False, na=False)), 'in_school']='Y'

x.seeking_work.unique()
x.loc[(x['seeking_work'].str.contains('N', case=False, na=False)) | (x['seeking_work'].isnull()), 'seeking_work']=0
x.loc[(x['seeking_work'].str.contains('Y', case=False, na=False)), 'seeking_work']=1
x.loc[(x['seeking_work'].str.contains('-', case=False, na=False)), 'seeking_work']=0

x.loc[(x['private_work']==1) | (x['public_work']==1) | (x['had_job'].str.contains('Y', case=False, na=False)), 'had_job']=1
x.loc[(x['had_job'].str.contains('N', case=False, na=False)) | (x['had_job'].str.contains('-', case=False, na=False)) | (x['had_job'].isnull()), 'had_job']=0

x['worker_type'].unique()
x.loc[(x['worker_type'].str.contains('O', case=False, na=False)) | (x['worker_type'].str.contains('-', case=False, na=False)) | (x['worker_type'].isnull()), 'worker_type']='X'
x.loc[(x['worker_type'].str.contains('H', case=False, na=False)), 'worker_type']='H'
x.loc[(x['had_job']==1), 'worker_type']='N'

#Key Tables
#Worker Types
wk = x['worker_type'].to_frame()
wk = wk.drop_duplicates(ignore_index=True)
wk.insert(0, 'work_id', range(1, 1+len(wk)))

conditions = [wk['worker_type']=='X', wk['worker_type']=='U', wk['worker_type']=='N', wk['worker_type']=='H', wk['worker_type']=='S']
categories = ['unknown', 'unable', 'employed', 'homemaker', 'student']
wk['worker_description'] = np.select(conditions, categories, default='Unknown')

#Resident Key
r = x[['first_name', 'last_name', 'sex', 'race']]
r.insert(0, 'resident_id', range(1, 1+len(r)))

#Street Key
st = x['street'].to_frame()
st = st.drop_duplicates(ignore_index=True)

st.insert(0, 'street_id', range(1, 1+len(st)))

#Marital Status Key
m = x['marital_status'].to_frame()
m = m.drop_duplicates(ignore_index=True)

m.insert(0, 'marital_id', range(1, 1+len(m)))

conditions = [m['marital_status']=='S', m['marital_status']=='M', m['marital_status']=='W', m['marital_status']=='D', m['marital_status']=='U']
categories = ['Single', 'Married', 'Widowed', 'Divorced', 'Unknown']
m['description'] = np.select(conditions, categories, default='Unknown')

#Household Title Key
h = x['relationship_to_household'].to_frame()
h = h.drop_duplicates(ignore_index=True)
h.insert(0, 'household_id', range(1, 1+len(h)))

#Now let's port it into MySQL
connection = mysql.connector.connect(host = 'localhost', user = 'root', password = 'Purecolour1453!')

cursor = connection.cursor()
cursor.execute('drop database if exists census')
cursor.execute('create database census')

engine = create_engine('mysql+mysqlconnector://root:Password!@localhost/census')

r.to_sql('resident_key', con=engine, if_exists='append', index=False)
wk.to_sql('work_key', con=engine, if_exists='append', index=False)
st.to_sql('street_key', con=engine, if_exists='append', index=False)
m.to_sql('marital_key', con=engine, if_exists='append', index=False)
h.to_sql('household_key', con=engine, if_exists='append', index=False)

#Now let's 'join' the foreign keys ID columns to the parent table
a = x.merge(wk[['worker_type','work_id']], how='left', on=['worker_type'])
a.drop(columns=['worker_type'])

b = x.merge(st[['street','street_id']], how='left', on=['street'])
b.drop(columns=['street'])

c = x.merge(m[['marital_status','marital_id']], how='left', on=['marital_status'])
c.drop(columns=['street'])

d = x.merge(h[['relationship_to_household', 'household_id']], how='left', on=['relationship_to_household'])
d.drop(columns=['relationship_to_household'])

g = x.merge(r[['first_name', 'last_name', 'resident_id']], how='left', on=['first_name', 'last_name'])
g.drop(columns=['first_name', 'last_name', 'sex', 'race'])
{% endhighlight %}
