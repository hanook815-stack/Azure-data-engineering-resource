this we need to create in Azure Database
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
