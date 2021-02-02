# U.S. Immigration Data
### Data Engineering Capstone Project

#### Project Summary
In this project, we will develop a data model designed for Online Analytical Processing (OLAP) to support queries analyzing U.S. immigration data. In the data model, we complemented the U.S. immigration data with U.S. cities' demographics data (ETL'ed in a fact table) as well as dimension tables for arrival ports, countries, visa types, etc. Considering: a) the Star Schema for the data model, which is optimized for OLAP queries, and b) a large amount of data, the prospects for analysis and extracting insights from the data are endless. We will touch on Exploratory Data Analysis on the collected data.

The project follows the following steps:
* Step 1: Scope the Project and Gather Data
* Step 2: Explore and Assess the Data
* Step 3: Define the Data Model
* Step 4: Run ETL to Model the Data
* Step 5: Conclusion and future work


#### Scope 
We'll utilize Pyspark to perform an extract, transform, load (ETL) pipeline. Considering that the data formats are inconsistent across all data sets, which resembles an ETL from a data lake, the extracted data will be cleaned and used to build the Star Schema in the data model as Apache Parquet columnar format optimized for OLAP queries.

We can use Spark to concatenate data in each path into a single data frame. However, we'll opt for using only a single path since: 
1. its size meets the requirement for this project, 
2. the data model and ETL pipeline is expected to work the same as we scale up the size of the data, 
3. it's easier to identify the errors and verify our theory on a small set of data before scaling up.


#### Datasets 

* ##### Immigration Data
The immigration data is in a folder in this workspace and accessible on the following path: <code>../../data/18-83510-I94-Data-2016/</code>. There's a file for each month of the year. An example file name is i94_apr16_sub.sas7bdat. Each file has a three-letter abbreviation for the month name. So a full file path for June would look like this: <code>../../data/18-83510-I94-Data-2016/i94_jun16_sub.sas7bdat</code>. This data comes from the U.S. National Tourism and Trade Office.
More information on the immigration data [here](https://travel.trade.gov/research/reports/i94/historical/2016.html).

* ##### U.S. City Demographic Data
 This data comes from OpenSoft. You can read more about it [here](https://public.opendatasoft.com/explore/dataset/us-cities-demographics/export/).

* ##### Tools/Technologies Used
Python, Apache Spark, Pandas


#### Data Model

![data model star schema](star_schema.png)

#### ETL Data Pipelines
the steps necessary to pipeline (ingest, clean, populate tables) the data into the chosen data model:
1. load US [immegration data](https://travel.trade.gov/research/reports/i94/historical/2016.html) and [demographic data](https://public.opendatasoft.com/explore/dataset/us-cities-demographics/export/) from sources.
2. Extract label data from the SAS label description file, excluding unnecessary spaces and characters.
3. Further clean the ports' data by splitting the cities from states.
4. Create dimension table from cleaned data extracted from SAS label description file. 
5. Investigate for duplicate rows and missing values, drop if necessary.
6. Create facts and dimensions table by joining immigration, demographic tables, and corresponding columns in dimension tables.
7. Perform data quality checks on fact and dimension tables.
8. Save dataframes into Apache Parquet columnar format.

#### Data Dictionary

<code>i_94_fact_table</code>
- **cicid:** id from sas file
- **entry_year:** 4 digit year
- **entry_month:** numeric month
- **origin_country_code:** i94 country code as per SAS Labels Descriptions file
- **port_code:** i94port code as per SAS Labels Descriptions file
- **arrival_date:** date of arrival in U.S.
- **travel_mode_code:** code for travel mode of arrival as per SAS Labels Descriptions file
- **us_state_code:** two letter U.S. state code
- **departure_date:** departure date from U.S.
- **age:** age of the immigrant
- **visa_category_code:** visa category code as per SAS Labels Descriptions file
- **occupation:** occupation of immigrant
- **gender:** gender of immigrant
- **birth_year:** birth year of immigrant
- **entry_date:** Date to which admitted to U.S. (allowed to stay until)
- **airline:** airline code used to arrive in U.S.
- **admission_number:** admission number
- **flight_number:** flight number
- **visa_type:** visa type

<code>dim_city_demographics</code>
- **port_code:** i94port code
- **city:** Common name of U.S. city. Part of the candidate key to airports
- **state_code:** two letter U.S. sate code
- **male_population:** total male population
- **female_population:** total female population
- **total_population:** total population
- **number_of_veterans:** number of veterans
- **num_foreign_born:** number of foreign born

#### Conclusion and Reflection

The target star Schema has been successfully created and passed the quality checks. The data model employed the tool and technologies: python, Pandas, and Apache Spark. The Schema is optimized to support OLAP queries. Even though the data model is created using a portion of the data, it is expected to work when we concatenate all of the data and load it with Spark. However, with a larger dataset or handling a large number of simultaneous queries, using more clusters or EMR would as the size of the data scales up.

#### Tools and Technologies usage:

Python, Pandas, and Spark enabled us to read, clean, ingest, process, load, and store the data with ease. 

Spark was chosen over Hadoop for its better flexibility in wrangling data and better adaption in working on both local machines and scaling up nicely and efficiently on a larger cluster of nodes. 

Pandas dataframes are extremely easy to explore and analyze. If we take a portion of the data in the spark session and put it into Pandas dataframe, we can implement a plethora of built-in functions that can help for serving as a proof of concept on a smaller scale, then test it on a larger scale with spark dataframes and user-defined functions.

We could use more technologies in future work. We might add Apache Airflow DAGs to automate fact table updates and data quality checks. Cloud-based storage and/or instance can be utilized to ensure that database is always up.

#### Data Model:
The star schema was designed with OLAP queries in mind. The ETL process, and sequentially the data quality checks, ensured that the dimension tables are normalized. This normalization allows for analytical queries they are more intuitive and less complex, and computationally inexpensive in case of aggregation on any of the dimensions. And due to the fact that the dimension tables are relatively small with a sufficiently large number of uniques primary keys, the joins are also intuitive and less taxing on the database.

* **When the data was increased by 100x**
We might consider deploying more work nodes for Spark with data distributed between them. If this might not suffice as data scales up, we could use instances of EC2 clusters for more processing power and Amazon RedShift S3 buckets for storage, or Amazon EMR, albeit these cloud-based options are more costly than the addition of more nodes in the cluster.

* **How do you run this pipeline on a daily basis by 7 am every day.**
In this case, we can design Apache Airflow DAGs that execute the ETL pipeline as well as data quality checks, scheduled to run every day at 7 am. The DAG structure should be well-thought-out that it has proper error handling with descriptive logging messages so we can better understand and trace errors in the data lineage.

* **How do you make your database could be accessed by 100+ people?**
In this case, we might consider creating additional tables that are optimized for the most frequent queries. However, if queries are unpredictable, we can opt to host the data in the cloud. Options include AWS RDS, and MS Azure
