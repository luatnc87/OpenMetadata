---
title: Profiler Workflow
slug: /connectors/ingestion/workflows/profiler
---

# Profiler Workflow

Learn how to configure and run the Profiler Workflow to extract Profiler data and execute the Data Quality.

## UI configuration
After the metadata ingestion has been done correctly, we can configure and deploy the Profiler Workflow.

This Pipeline will be in charge of feeding the Profiler tab of the Table Entity, as well as running any tests configured in the Entity.

{% image
  src="/images/v1.0/features/ingestion/workflows/profiler/profiler-summary-table.png"
  alt="Table profile summary page"
  caption="Table profile summary page"
 /%}


{% image
  src="/images/v1.0/features/ingestion/workflows/profiler/profiler-summary-colomn.png"
  alt="Column profile summary page"
  caption="Column profile summary page"
 /%}



### 1. Add a Profiler Ingestion
From the Service Page, go to the Ingestions tab to add a new ingestion and click on Add Profiler Ingestion.

{% image
  src="/images/v1.0/features/ingestion/workflows/profiler/add-profiler-workflow.png"
  alt="Add a profiler service"
  caption="Add a profiler service"
 /%}


### 2. Configure the Profiler Ingestion
Here you can enter the Profiler Ingestion details.

{% image
  src="/images/v1.0/features/ingestion/workflows/profiler/configure-profiler-workflow.png"
  alt="Set profiler configuration"
  caption="Set profiler configuration"
 /%}


#### Profiler Options
**Name**
Define the name of the Profiler Workflow. While we only support a single workflow for the Metadata and Usage ingestion, users can define different schedules and filters for Profiler workflows.

As profiling is a costly task, this enables a fine-grained approach to profiling and running tests by specifying different filters for each pipeline.

**Database filter pattern (Optional)**
regex expression to filter databases.

**Schema filter pattern (Optional)**
regex expression to filter schemas.

**Table filter pattern (Optional)**
regex expression to filter tables.

**Profile Sample (Optional)**
Set the sample to be use by the profiler for the specific table.
- `Percentage`: Value must be between 0 and 100 exclusive (0 < percentage < 100). This will sample the table based on a percentage
- `Row Count`: The table will be sampled based on a number of rows (i.e. `1,000`, `2,000`), etc.

**Auto PII Tagging (Optional)**
Configuration to automatically tag columns that might contain sensitive information.

- **Confidence (Optional)**
If `Auto PII Tagging` is enable, this confidence level will determine the threshold to use for OpenMetadata's NLP model to consider a column as containing PII data.

**Thread Count (Optional)**
Number of thread to use when computing metrics for the profiler. For Snowflake users we recommend setting it to 1. There is a known issue with one of the dependency (`snowflake-connector-python`) affecting projects with certain environments. 

**Timeout in Seconds (Optional)**
This will set the duration a profiling job against a table should wait before interrupting its execution and moving on to profiling the next table. It is important to note that the profiler will wait for the hanging query to terminiate before killing the execution. If there is a risk for your profiling job to hang, it is important to also set a query/connection timeout on your database engine. The default value for the profiler timeout is 12-hours.

**Ingest Sample Data**
Whether the profiler should ingest sample data

### 3. Schedule and Deploy
After clicking Next, you will be redirected to the Scheduling form. This will be the same as the Metadata and Usage Ingestions. Select your desired schedule and click on Deploy to find the usage pipeline being added to the Service Ingestions.

### 4. Updating Profiler setting at the table level
Once you have created your profiler you can adjust some behavior at the table level by going to the table and clicking on the profiler tab 

{% image
  src="/images/v1.0/features/ingestion/workflows/profiler/accessing-table-profile-settings.png"
  alt="table profile settings"
  caption="table profile settings"
 /%}

{% image
  src="/images/v1.0/features/ingestion/workflows/profiler/table-profile-summary-view.png"
  alt="table profile settings"
  caption="table profile settings"
 /%}


#### Profiler Options
**Profile Sample**
Set the sample to be use by the profiler for the specific table.
- `Percentage`: Value must be between 0 and 100 exclusive (0 < percentage < 100). This will sample the table based on a percentage
- `Row Count`: The table will be sampled based on a number of rows (i.e. `1,000`, `2,000`), etc.

**Profile Sample Query**
Use a query to sample data for the profiler. This will overwrite any profle sample set.

**Enable Column Profile**
This setting allows user to exclude or include specific columns and metrics from the profiler.

*Note: for Google BigQuery tables partitioned on timestamp/datetime column type, month and year interval are not supported. You will need to set the `Interval Unit` to `DAY` or `HOUR`.*

**Enable Partition**
If your table includes a timestamp, date or datetime column type you can enable partitionning. If enabled, the profiler will fetch the last `<interval>` `<interval unit>` of data to profile the table. Note that if "profile sample" is set, this configuration will be used against the partitioned data and not the whole table.
- `Column Name`: this is the name of the column that will be used as the partition field
- `Interval Type`:
  - `TIME-UNIT`: a business logical timestamp/date/datetime (e.g. order date, sign up datetime, etc.)
  - `INGESTION-TIME`: a process logical timestamp/date/datetime (i.e. when was my data ingested in my table)
- `Interval`: the interval value (e.g. `1`, `2`, etc.)
- `Interval Unit`: 
  - `HOUR`
  - `DAY`
  - `MONTH`
  - `YEAR`


## YAML Configuration

In the [connectors](/connectors) section we showcase how to run the metadata ingestion from a JSON file using the Airflow SDK or the CLI via metadata ingest. Running a profiler workflow is also possible using a JSON configuration file.

This is a good option if you which to execute your workflow via the Airflow SDK or using the CLI; if you use the CLI a profile workflow can be triggered with the command `metadata profile -c FILENAME.yaml`. The `serviceConnection` config will be specific to your connector (you can find more information in the [connectors](/connectors) section), though the sourceConfig for the profiler will be similar across all connectors.

```yaml

  [...]
  sourceConfig:
    config:
      type: Profiler
      generateSampleData: true
      profileSample: 60
      profileSampleType: ROWS
      #profileSampleType: PERCENTAGE
      databaseFilterPattern: 
        includes: 
          - dev
      schemaFilterPattern:
        includes: 
          - dbt_jaffle
      tableFilterPattern:
        includes: 
          - orders
          - customers
  [...]
```

 ## Profiler Best Practices
 When setting a profiler workflow it is important to keep in mind that queries will be running against your database. Depending on your database engine, you may incur costs (e.g., Google BigQuery, Snowflake). Execution time will also vary depending on your database engine computing power, the size of the table, and the number of columns. Given these elements, there are a few best practices we recommend you follow.

 ### 1. Profile what you Need
 Profiling all the tables in your data platform might not be the most optimized approach. Profiled tables give an indication of the structure of the table, which is most useful for tables where this information is valuable (e.g., tables used by analysts or data scientists, etc.).

 When setting up a profiler workflow, you have the possibility to filter out/in certain databases, schemas, or tables. Using this feature will greatly help you narrow down which table you want to profile.

 ### 2. Sampling and Partitionning your Tables
 On a table asset, you have the possibility to add a sample percentage/rows and a partitioning logic. Doing so will significantly reduce the amount of data scanned and the computing power required to perform the different operations. 

 For sampling, you can set a sampling percentage at the workflow level.

 ### 3. Excluding/Including Specific Columns/Metrics
 By default, the profiler will compute all the metrics against all the columns. This behavior can be fine-tuned only to include or exclude specific columns and specific metrics.

 For example, excluding `id` columns will reduce the number of columns against which the metrics are computed.

 ### 4. Set Up Multiple Workflow
 If you have a large number of tables you would like to profile, setting up multiple workflows will help distribute the load. It is important though to monitor your instance CPU, and memory as having a large amount of workflow running simultaneously will require an adapted amount of resources.