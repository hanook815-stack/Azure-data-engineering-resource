1.Github witgh required data in CSV format
2.create Azure resource
   -- create ADF(Azure data factory) -- we need to create Linked service for Azure SQL database and HTTP( to get the data from GitHub and          provide base URL : https://raw.githubusercontent.com/) 
      Then create new pipeline and set the and click on source and provide relative URL
          Relaticve URL hanook815-stack/Azure-data-engineering-resource/refs/heads/main/SalesData.csv
          and then click on Advanced and open the dataset copy and click on parameters name it as Load_Flag
          then go to connection and copy relative URL and clikc on dynamic contect paste it in box hanook815-stack/Azure-data-engineering-resource/refs/heads/main/@{dataset().load_flg}
          and under source provide load_flag value as SalesData.csv
          Then click on Sink data set and create connection tha we need to store the data ( Azure SQL database)                                                                             
   -- create Azure sql database
   -- create ADLS2 to store the data in the form of Bronze , Silver and Gold   

This we need to create in Azure Database to store the data which is coming from CSV files
=====================
create table source_cars_data(Branch_ID VARCHAR(200),
Dealer_ID VARCHAR(200),
Model_ID VARCHAR(200),
Revenue BIGINT,
Units_Sold BIGINT,
Date_ID VARCHAR(200),
Day INT,
Month INT,
Year INT,
BranchName VARCHAR(200),
DealerName VARCHAR(200),
Product_Name VARCHAR(200))

select * from source_cars_data

create table watermark_table
( 
  last_load VARCHAR(2000)
)

select * from watermark_table

select min(Date_ID) from dbo.source_cars_data

INSERT INTO watermark_table VALUES ('DT00000')

select * from dbo.source_cars_data where Date_ID > 'DT00000'

select * from dbo.source_cars_data

Store Procedure
===========

CREATE PROCEDURE UpdateWaterMarktable
@lastload VARCHAR(2000)
AS
BEGIN
   -- Start the transaction
    BEGIN TRANSACTION;
    -- update the incremental column in the table
    UPDATE watermark_table
    SET last_load = @lastload
    COMMIT TRANSACTION;
    END;

Below are the pipeline setting in ADF
=============================

    last_load -- > setting --> provide table name as watermark_table and provide the Query select * from watermark_table
    current_load --> setting > provide table name as source_cars_date and provide Query select * from watermark_table

The below Query we need to set it under Copy_increment_date
=========================================
select * from source_cars_data where Date_ID > '@{activity('last_load').output.value[0].last_load}' AND Date_ID <= '@{activity('current_load').output.value[0].maxdate}'

and provide the table name as well source_cars_data

@activity('current_load').output.value[0].maxdate --- This valu we need to set it under watermark_update


Create Databricks 
