# Working with Spark Dataframes 

## Introduction

If you come from the SQL world, you may be unused to the mechnics of using dataframes in Spark. 
This document will get your productive fast. Just follow the instructions be and by the end you'll be able
to do basic dataframe manipulations.

## Preparing

You'll need
- an Azure Synapse Analytics workspace. 
- to be assigned the Workspace Admin role in the Synpse Workspace.
- to have contributor access to the Workspace (done via the Azure portal)
- read and write permissions to an ADLSGEN2 account

## Spark pool

If you don't already have, one create a spark pool. For this document, we'll assume it has a name of **spark1**.

## Create a notebook and run a Hello World

Create an new notebook. 
Add a new cell.
In the cell put this code

```
%%pyspark
print("Hello World!")
```

This notebook does almost nothing, but running it ensuresthat the worksapce is configured correctly at some minumum level.

## A note about cell magics

You see that the first line in the cell is `%%pyspark` this is called a **cell magic**. Notebooks have a default language, and this magic override the language in the that cell. We'll use cell magics in this doc because we'll be mixing languages often. 

The specific magic we used indicates that this cell will use the Python language to work with spark.

## Manually creating a dataframe

For debugging and learning, it's very useful to have a dataset available that:
* doesn't require permissions to access
* is guartanteed to always be available to the notebook
* is not too large

What we'll learn now is how to build a dataframe manually in python and then render that dataframe.


```
%%pyspark

df = spark.createDataFrame(
    [
        (1, 'foo'), 
        (2, 'bar'),
    ],
    ['id', 'txt'] )

df.show()
```

It should be pretty obvious what is going on here. We are building a dataframe df adding the rows via python code.

Calling df.show() causes the output to be seen in the cell output. You should see:

```
+---+---+
| id|txt|
+---+---+
|  1|foo|
|  2|bar|
+---+---+
```


## Finding out the schema of a dataframe


```
%%pyspark
df.printSchema()
```

you'll see:

```
root
 |-- id: long (nullable = true)
 |-- txt: string (nullable = true)
```


## Creating a dataframe with types

Sometimes it's very useful to be specific about the schemas involved. You can do that also

```
import pyspark.sql.types as sqltypes

data = [
        (1, 'foo', True, '2019-10-15T11:13:04Z'), 
        (2, 'bar', True, '2019-10-15T11:51:43Z'),
]

schema = sqltypes.StructType([
    sqltypes.StructField('id', sqltypes.IntegerType(), True),
    sqltypes.StructField('name', sqltypes.StringType(), True),
    sqltypes.StructField('open', sqltypes.BooleanType(), True),
    sqltypes.StructField('open', sqltypes.StringType(), True)
])

df = spark.createDataFrame(data, schema)
df.show()
df.printSchema()
```

There's no direct way to create datetime values, so we use StringType for now.

## Converting strings to datetimes


```
import pyspark.sql.types as sqltypes

data = [
        (1, 'foo', True, '2019-10-15', '2019-10-15T11:13:04Z'), 
        (2, 'bar', True, '2019-10-15', '2019-10-15T11:51:43Z'),
]

schema = sqltypes.StructType([
    sqltypes.StructField('id', sqltypes.IntegerType(), True),
    sqltypes.StructField('name', sqltypes.StringType(), True),
    sqltypes.StructField('open', sqltypes.BooleanType(), True),
    sqltypes.StructField('startdate', sqltypes.StringType(), True),
    sqltypes.StructField('startts', sqltypes.StringType(), True)
])

df = spark.createDataFrame(data, schema)

def col_to_type(df_, colname, t):
    df_ = df_.withColumn("NewCol__", df[colname].cast(t))
    df_ = df_.drop(colname)
    df_ = df_.withColumnRenamed("NewCol__",colname)
    return df_

df = col_to_type(df, "startdate", sqltypes.DateType() )
df = col_to_type(df, "startts", sqltypes.TimestampType() )

df.show()
df.printSchema()
```

```
+---+----+----+----------+-------------------+
| id|name|open| startdate|            startts|
+---+----+----+----------+-------------------+
|  1| foo|true|2019-10-15|2019-10-15 11:13:04|
|  2| bar|true|2019-10-15|2019-10-15 11:51:43|
+---+----+----+----------+-------------------+

root
 |-- id: integer (nullable = true)
 |-- name: string (nullable = true)
 |-- open: boolean (nullable = true)
 |-- startdate: date (nullable = true)
 |-- startts: timestamp (nullable = true)
```






