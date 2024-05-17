# Data Pipeline Flowchart

![image](https://github.com/dna737/tokyo-olympics-DE/assets/102070227/9ca0771e-d162-4193-b36b-0bc86d8acc49)


## 1. Data source
I got my dataset from [Kaggle](https://www.kaggle.com/datasets/arjunprasadsarkhel/2021-olympics-in-tokyo). It didn't require any cleaning process, so it was very easy to get started with it.

## 2. Data ingestion
I then used the Azure Data Factory to move the data from the source to Azure Data Lake Storage (ADLS).

![image](https://github.com/dna737/tokyo-olympics-DE/assets/102070227/0b64a055-dc75-4ffc-a901-68962f8ef9ed)

## 3. Raw Data Storage
Using the previous step, I was able to store the raw data in ADLS.

![image](https://github.com/dna737/tokyo-olympics-DE/assets/102070227/af44901f-7482-4d3e-92ff-559f1ead6665)

## 4. Transformation
Then, in Azure Databricks, I mounted the Data Lake Storage and applied transformations as follows:

### Mounting the DLS:
```
tenantID = dbutils.secrets.get(scope="kev-vault-scope", "tenantID")
storageID = dbutils.secrets.get(scope="kev-vault-scope", "storageID")
dataID = dbutils.secrets.get(scope="kev-vault-scope", "dataID")
configs = {"fs.azure.account.auth.type": "OAuth",
"fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
"fs.azure.account.oauth2.client.id": dbutils.secrets.get(scope="kev-vault-scope", "clientID"),
"fs.azure.account.oauth2.client.secret": dbutils.secrets.get(scope="kev-vault-scope", "secretKey"),
"fs.azure.account.oauth2.client.endpoint": f'https://login.microsoftonline.com/{tenantID}/oauth2/token'}

dbutils.fs.mount(
source = f'abfss://{dataID}a@{storageID}.dfs.core.windows.net', # container@storageacc
mount_point = "/mnt/tokyoolympics",
extra_configs = configs)
```

### Transforming the data:
```
athletes = spark.read.format("csv").option("header","true").option("inferSchema","true").load("/mnt/tokyoolympics/raw-data/athletes.csv")
coaches = spark.read.format("csv").option("header","true").option("inferSchema","true").load("/mnt/tokyoolympics/raw-data/coaches.csv")
entriesgender = spark.read.format("csv").option("header","true").option("inferSchema","true").load("/mnt/tokyoolympics/raw-data/entriesGender.csv")
medals = spark.read.format("csv").option("header","true").option("inferSchema","true").load("/mnt/tokyoolympics/raw-data/medals.csv")
teams = spark.read.format("csv").option("header","true").option("inferSchema","true").load("/mnt/tokyoolympics/raw-data/teams.csv")
```

### Modifying the Schema to interpret the correct data type:
```
entriesgender = entriesgender.withColumn("Female",col("Female").cast(IntegerType()))\
    .withColumn("Male",col("Male").cast(IntegerType()))\
    .withColumn("Total",col("Total").cast(IntegerType()))
```

### Storing the transformed data in the appropriate location:
```
athletes.repartition(1).write.mode("overwrite").option("header",'true').csv("/mnt/tokyoolympics/transformed-data/athletes")
coaches.repartition(1).write.mode("overwrite").option("header","true").csv("/mnt/tokyoolympics/transformed-data/coaches")
entriesgender.repartition(1).write.mode("overwrite").option("header","true").csv("/mnt/tokyoolympics/transformed-data/entriesGender")
medals.repartition(1).write.mode("overwrite").option("header","true").csv("/mnt/tokyoolympics/transformed-data/medals")
teams.repartition(1).write.mode("overwrite").option("header","true").csv("/mnt/tokyoolympics/transformed-data/teams")
```

## 5. Transformed Data Storage:
We now have the transformed data as follows:
![image](https://github.com/dna737/tokyo-olympics-DE/assets/102070227/96eb1af1-09ab-414e-bcc4-ebca2d554d7d)

## 6. Running SQL queries using Azure Synapse Analytics:

We can now load the transformed data in a database and start performing SQL operations on it!

![image](https://github.com/dna737/tokyo-olympics-DE/assets/102070227/0813071e-42fb-45e3-a638-533f21b38588)

Thanks for reading!

