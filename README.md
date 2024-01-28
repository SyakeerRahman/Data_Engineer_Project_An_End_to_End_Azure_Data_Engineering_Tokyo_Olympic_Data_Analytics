# Data_Engineer_Project_An_End_to_End_Azure_Data_Engineering_Tokyo_Olympic_Data_Analytics

<img width="527" alt="image" src="https://github.com/SyakeerRahman/Data_Engineer_Project_An_End_to_End_Azure_Data_Engineering_Tokyo_Olympic_Data_Analytics/assets/105381652/0559a4ca-14bb-4978-8ac6-ee917e9bebf9">

## Data Source and Resources Setup

1.  create resource group and then create these resources inside it:
  -  data factory
  -  data lake gen2
  -  databricks
  -  synapse

## Data Ingestion Using Azure Data Factory into Azure Data Lake

1. inside azure data lake create a container and create 2 folder named as raw-data and transformed-data
<img width="572" alt="image" src="https://github.com/SyakeerRahman/Data_Engineer_Project_An_End_to_End_Azure_Data_Engineering_Tokyo_Olympic_Data_Analytics/assets/105381652/96251cbc-03c9-4a75-b91e-917bee480900">

2. open data factory and create a pipeline, choose copy activity and use http as source
<img width="576" alt="image" src="https://github.com/SyakeerRahman/Data_Engineer_Project_An_End_to_End_Azure_Data_Engineering_Tokyo_Olympic_Data_Analytics/assets/105381652/4f4359e2-a091-4c99-a458-16157bf4e0af">

3. as for sink, the destination is adls inside container raw
<img width="587" alt="image" src="https://github.com/SyakeerRahman/Data_Engineer_Project_An_End_to_End_Azure_Data_Engineering_Tokyo_Olympic_Data_Analytics/assets/105381652/477fffa1-5645-43f6-afe3-28933c80ad6e">

4. repeat step 2 & 3 for other file as well
<img width="593" alt="image" src="https://github.com/SyakeerRahman/Data_Engineer_Project_An_End_to_End_Azure_Data_Engineering_Tokyo_Olympic_Data_Analytics/assets/105381652/ce2d6268-a2da-4923-8464-8885f007d248">


## Data Transformation using Azure Databricks

1. open azure databricks and create a new compute

2. create a new workbook and do some checking and transformation using python
```
from pyspark.sql.functions import col
from pyspark.sql.types import IntegerType, DoubleType, BooleanType, DateType


configs = {"fs.azure.account.auth.type": "OAuth",
"fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
"fs.azure.account.oauth2.client.id": "",
"fs.azure.account.oauth2.client.secret": '',
"fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/tanent_id/oauth2/token"}


dbutils.fs.mount(
source = "abfss://tokyo-olympic-data@tokyoolympicdata.dfs.core.windows.net", # contrainer@storageacc
mount_point = "/mnt/tokyoolymic",
extra_configs = configs)


%fs
ls "/mnt/tokyoolymic"


spark

athletes = spark.read.format("csv").option("header","true").option("inferSchema","true").load("/mnt/tokyoolymic/raw-data/athletes.csv")
coaches = spark.read.format("csv").option("header","true").option("inferSchema","true").load("/mnt/tokyoolymic/raw-data/coaches.csv")
entriesgender = spark.read.format("csv").option("header","true").option("inferSchema","true").load("/mnt/tokyoolymic/raw-data/entriesgender.csv")
medals = spark.read.format("csv").option("header","true").option("inferSchema","true").load("/mnt/tokyoolymic/raw-data/medals.csv")
teams = spark.read.format("csv").option("header","true").option("inferSchema","true").load("/mnt/tokyoolymic/raw-data/teams.csv")
     
athletes.show()

athletes.printSchema()

coaches.show()

coaches.printSchema()

entriesgender.show()

entriesgender.printSchema()

entriesgender = entriesgender.withColumn("Female",col("Female").cast(IntegerType()))\
    .withColumn("Male",col("Male").cast(IntegerType()))\
    .withColumn("Total",col("Total").cast(IntegerType()))


entriesgender.printSchema()

medals.show()

medals.printSchema()

teams.show()

teams.printSchema()

# Find the top countries with the highest number of gold medals
top_gold_medal_countries = medals.orderBy("Gold", ascending=False).select("Team_Country","Gold").show()

# Calculate the average number of entries by gender for each discipline
average_entries_by_gender = entriesgender.withColumn(
    'Avg_Female', entriesgender['Female'] / entriesgender['Total']
).withColumn(
    'Avg_Male', entriesgender['Male'] / entriesgender['Total']
)
average_entries_by_gender.show()

athletes.repartition(1).write.mode("overwrite").option("header",'true').csv("/mnt/tokyoolymic/transformed-data/athletes")
coaches.repartition(1).write.mode("overwrite").option("header","true").csv("/mnt/tokyoolymic/transformed-data/coaches")
entriesgender.repartition(1).write.mode("overwrite").option("header","true").csv("/mnt/tokyoolymic/transformed-data/entriesgender")
medals.repartition(1).write.mode("overwrite").option("header","true").csv("/mnt/tokyoolymic/transformed-data/medals")
teams.repartition(1).write.mode("overwrite").option("header","true").csv("/mnt/tokyoolymic/transformed-data/teams")
     
```   

## Data Analytics using Azure Synapse

1. create synapse workspace and integrate new external table
<img width="500" alt="image" src="https://github.com/SyakeerRahman/Data_Engineer_Project_An_End_to_End_Azure_Data_Engineering_Tokyo_Olympic_Data_Analytics/assets/105381652/04b81358-1faa-4406-8d1f-334a8d881a90">

2. write query
<img width="425" alt="image" src="https://github.com/SyakeerRahman/Data_Engineer_Project_An_End_to_End_Azure_Data_Engineering_Tokyo_Olympic_Data_Analytics/assets/105381652/483fca18-09e4-4dab-aed3-d923810df8ac">


## Data Reporting and Dashboard 
