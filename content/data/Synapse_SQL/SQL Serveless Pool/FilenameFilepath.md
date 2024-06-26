## Synapse Serverless SQL pool 

[Back<](https://github.com/LiliamLeme/FTALive-Sessions_Synapse_SQL/blob/main/content/data/Synapse_SQL/SQL%20Serveless%20Pool/SynapseCETAS.md) -[Home](https://github.com/LiliamLeme/FTALive-Sessions_Synapse_SQL/blob/main/content/data/Synapse_SQL/SQL%20Serveless%20Pool/Agenda_serveless.md)\- [>Next ](https://github.com/LiliamLeme/FTALive-Sessions_Synapse_SQL/blob/main/content/data/Synapse_SQL/SQL%20Serveless%20Pool/SQL%20serverless%20pool%20and%20Spark%20Integration.md)

### FileName and FilePath 

You can use function `filepath` and `filename` to return file names and/or the path in the result set. Or you can use them to filter data based on the file name and/or folder path. These functions are described in the syntax section [filename function](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/query-data-storage#filename-function) and [filepath function](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/query-data-storage#filepath-function). In the following sections, you'll find short descriptions along samples.

**Example**: https://raw.githubusercontent.com/Azure-Samples/Synapse/main/SQL/Samples/LdwSample/SampleDB.sql

### Filename

This function returns the file name that row originates from.

The following sample reads the NYC Yellow Taxi data files for the last three months of 2017 and returns the number of rides per file. The OPENROWSET part of the query specifies which files will be read.

```sql
SELECT
    nyc.filename() AS [filename]
    ,COUNT_BIG(*) AS [rows]
FROM  
    OPENROWSET(
        BULK 'parquet/taxi/year=2017/month=9/*.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) nyc
GROUP BY nyc.filename();
```

The following example shows how *filename()* can be used in the WHERE clause to filter the files to be read. It accesses the entire folder in the OPENROWSET part of the query and filters files in the WHERE clause.

Your results will be the same as the prior example.

```SELECT
    SELECT
    r.filename() AS [filename]
    ,COUNT_BIG(*) AS [rows]
FROM OPENROWSET(
    BULK 'csv/taxi/yellow_tripdata_2017-*.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV',
        PARSER_VERSION = '2.0',
        FIRSTROW = 2) 
        WITH (C1 varchar(200) ) AS [r]
WHERE
    r.filename() IN ('yellow_tripdata_2017-10.csv', 'yellow_tripdata_2017-11.csv', 'yellow_tripdata_2017-12.csv')
GROUP BY
    r.filename()
ORDER BY
    [filename];
```

### Filepath

The filepath function returns a full or partial path:

- When called without a parameter, it returns the full file path that the row originates from. When DATA_SOURCE is used in OPENROWSET, it returns path relative to DATA_SOURCE.
- When called with a parameter, it returns part of the path that matches the wildcard on the position specified in the parameter. For example, parameter value 1 would return part of the path that matches the first wildcard.

```sql
SELECT
    r.filepath() AS filepath
    ,COUNT_BIG(*) AS [rows]
FROM OPENROWSET(
        BULK 'csv/taxi/yellow_tripdata_2017-1*.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV',
        PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        vendor_id INT
    ) AS [r]
GROUP BY
    r.filepath()
ORDER BY
    filepath;
```

### Partition Elimination
The usage of those functions with a partition set of folders enables serverless SQL Pool the possibility of using partition elemination to optimize the query results. Partition elimination means the optimizer will use the query filter to eliminate the partitions that do not satisfy it in early stages of the query and it will keep only the ones that has the results corresponding to the predicate.

#### Reference:

[Using file metadata in queries - Azure Synapse Analytics | Microsoft Learn](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/query-specific-files)

[Synapse Serverless SQL Pool - Performance and cost optimization with partitioning - Microsoft Community Hub](https://techcommunity.microsoft.com/t5/fasttrack-for-azure/synapse-serverless-sql-pool-performance-and-cost-optimization/ba-p/3673286)
