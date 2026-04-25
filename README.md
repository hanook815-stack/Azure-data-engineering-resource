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


Create Databricks -- and launch databricks
Workspace -- top of the hirarchy we can store the notbooks.
catalog -- manage databse,shemas , tables and volumes (Semi-structure data) etc
workflow -- ETL framework ,same as ADF
compute -- we can create clusters
Delta live table -- it is also like ETL flow

we will be creating Unity Metastore --  when we create unity metastore only then we can create compute because we get different kind of cluster when we enable unity catalog.
first we need to create Unity metastore then we will attach our databricks workspace to unity metastore.

click on workspace on top of the right -- > under this tab we can see manage account button.
how we will get manage account , we need to provide ENTRA ID in admin databricks(accounts.azuredatabricks.net). (Azure-->Home-->EntraID-->users-->click on your name -->copy Hastag ID)
copy entraid and go to accounts.azuredatabricks.net --> then click on continue with microsoft Entra id and then paste Entra ID which is copied from Azure -- > once open console page
Users management -- > add user --> put email account and asign the role as admin ,then we will see manage account in Microsoft Azure Databricks

to create unity metastore (accounts.azuredatabricks.net) -->  catalog --> click on create metastore ( first we need to create access connector -- > go to resource group and search  access connector for databricks -- then create )

**create access connector**

<img width="393" height="409" alt="image" src="https://github.com/user-attachments/assets/02455c76-631b-498e-a1ce-c18e1977c41e" />

( this mainly for managing data ADLS and Databricks) -- Then add access connector using IAM to the ADLS ( ADLS --> IAM --> Add role assignment) --> search and select storage blob contributor -> clcik next -- > select manage identities then select memebers and attach access connector then review and create --> once create copy resource ID and paste user access connector in databricks

<img width="1358" height="583" alt="image" src="https://github.com/user-attachments/assets/637abd36-22f1-4aa7-bad7-586b96a76b48" />

once we create metasore in accounts.azuredatabricks.net , now i want attache workspace ,click on workspace and select workspace and attach newly created metstore to workspace --> for this simpl go to catalog --> select once catalog and open --> then click on workspace --> click on assign to workspace --> then select any workspace and click on assign --> click on enable.( we can assign multiple worspaces to one unity metastore)  

<img width="916" height="469" alt="image" src="https://github.com/user-attachments/assets/b5476f65-4912-40e2-a0d4-d591e2d4b74c" />

