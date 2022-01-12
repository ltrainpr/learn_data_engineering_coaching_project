## Summary of My Learn Data Engineering Project
In the 4th quarter of 2021 I completed the [Learn Data Engineering Project](https://learndataengineering.com/p/data-engineering-coaching).  This project was done using the AWS ecosystem.

### Description
1. First part of the project was to select data to use.  After discussion with coach we selected [Smart Home Dataset with weather Information](https://www.kaggle.com/taranvee/smart-home-dataset-with-weather-information) available in Kaggle.  I downloaded this dataset unto my local machine.
1. I uploaded this file to S3 through the S3 dashboard.  The file was over 100MB in size.
1. Used dbdiagram.io site/tools to do a relational database design for data: https://dbdiagram.io/d/616d6b01940c4c4eec9ad0d6
1. Used EC2 instance and Cloud9 online editor to write python script that would read csv file stored in S3, transfrom the data using Pandas dataframe, and then write the transformed data to a Postgres AWS RDS.  Python script used contains:
  1. [Pandas](https://pypi.org/project/psycopg2/) dataframes to read the csv file from S3.  The data was sanitized and enhanced once in dataframe:
    - Added `timeframe` column with the time converted from string to timestamp.
    - Invalid, empty/nil, and not a number (NaN) data was replaced with default zero numeric values.  This required NumPy library.
    - `cloudCover` column had float and string values. These values were cast to integers.
    - `house_id` column was added to associate/group energy consuming devices to a house/home.
  1. Panda Dataframe is written back into a csv and temporarily stored in Python StringIO buffer.  StringIO buffer was selected because it is a fast way to do a [bulk database insert with Psycopg2](https://naysan.ca/2020/05/09/pandas-to-postgresql-using-psycopg2-bulk-insert-performance-benchmark/).
  1. [psycopg2](https://pypi.org/project/psycopg2/) library was used to connect with AWS relational database service (RDS) and upload the csv from StringIO bufffer to a Postgres database.
1. The above step was done for both weather and energy usage data.  A script was created to select and transform weather data, and another script was created to select and transform energy usage data.  Result is weather, energy_consuming_device, and home tables in RDS.
1. In order to access data from RDS, an AWS Lambda was created that would execute a `SELECT` SQL query for weather and energy_consuming_device tables.
1. In order to access data from Lambda, an AWS API Gateway was created for weather and energy_consuming_device tables.  This allows for table data to be queried using an http or https GET request.  You can also pass timeframe parameters through request to scope the timeframe of the requested table data. JSON is response format from Lambda and API Gateway.
1. An ETL lambda that makes GET https requests to API Gateway was created.  This ETL lambda was created to simulate what a background job/worker could use to get **new** data from RDS.  For example, an AWS EventBridge job could call ETL lambda on a daily basis to get new data from RDS, transfrom the data, and then store a csv file back into S3.  Pandas dataframes were used to capture the new data and convert it to a new csv file stored in S3.  This new data csv, has an easily identifiable file name with current date and time.
1. An AWS Redshift data warehouse cluster was created.  Weather and Energy Consuming Device tables were created in Redshift.  Queries from Redshift dashboard UI were used to copy the timestamped csv files created.  These queries simulate what an additional Lambda would do in production.  In a production environment an additional Lambda would be created that would copy new data from csv into Redshift weather or energy consuming device tables.
1. AWS Quicksight data visualization service was used to connect to Redshift, read data from tables, and display it in a chart or graph.
