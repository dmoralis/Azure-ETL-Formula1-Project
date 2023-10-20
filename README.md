# Formula1 Azure ETL Project

This project is part of my course attendance about Azure Databricks, using the Udemy platform. There is a certificate in this link https://ude.my/UC-e54cd93d-2409-4334-8495-18a514c3c0d3, that is proof of completance. The aim of this project is to extract Formula 1 data from Ergast API (via manual download), transform them so they are ready for presentation queries and at last load them in a storage container for future use.

![f1_project_graph](https://github.com/dmoralis/AirflowETLWeatherProject/assets/56253720/c62296be-61a8-4df1-8d62-cb3f92a40941)

**The image source is from the Azure Databricks & Spark For Data Engineers (PySpark / SQL) Course by Ramesh Retnasamy**

## Technologies Used

- **Azure Databricks**
- **PySpark - SQL**
- **Azure Workspace**
- **Azure Resource Group**
- **Azure Data Lake Gen2**
- **Azure Data Factory**
- **Azure Delta Lake**
- **Unity Catalog**

## Data Overview

The tables of the Formula 1 dataset are:

- Circuits (CSV)
- Races (CSV)
- Constructors (Single Line JSON)
- Drivers (Single Line Nested JSON)
- Results (Single Line JSON)
- PitStops (Multi Line JSON)
- LapTimes (Split CSV Files)
- Qualifying (Split Multi Line JSON Files)

Each one of them has a descriptive name. For more information about these tables you can visit the official site's user guide: https://ergast.com/docs/f1db_user_guide.txt 

## Project Details

### Ingestion

**Azure Databricks** is used w/ a combination of Notebooks and a Cluster in order to extract the files that was uploaded manually in a **Raw** container and convert the column names from Camel Case to Snake Case where its needed. Then save them in a **Processed** container of the Data Lake Gen2, and use them to create production level tables that are saved in the **Presentation** container in the same data lake.

The languages that are used in the notebooks are **PySpark** and **SQL**, so we can extract the data from the Raw container and create from them a  **schema** w/ managed tables for the Processed container. Respectively transform the data from the Processed schema and create new tables in a Presentation **schema** w/ managed tables:

- race_results
    Columns: |race_year|race_name|race_date|circuit_location|driver_name|driver_number|driver_nationality|team|fastest_lap|position|race_time|points|file_date|race_id|created_date|
- driver_standings
    Columns: |race_year|driver_name|team|total_points|wins|rank|
- constructor_standings
    Columns: |team|race_year|total_points|wins|rank|
- calculated_race_results
    Columns: |race_year|team_name|driver_id|driver_name|race_id|position|points|calculated_points|created_date|updated_date|

The data is coming to the data lake incrementally every week, and it is discriminated between **Dimension** and **Fact** tables. The first one requires a **Full Load** as it is small and the reload is not so frequent, and the second ones requires **Incremental Load** because the full data may be big sized and the updates are frequent. We categorize as dimension tables the: Circuits, Races, Contructors and Drivers and as fact tables: Results, PitStops, LapTimes, Qualifying. It is important to notice that due to the incremental load we need to append two new columns in each Fact table: **create_date** and **update_date** in order to have control of the the data updates and be able to increment data in dynamic calculated tables. 

In order to make a full load of the new data every week we **overwrite** the old data, and in order to incrementally add the new data to the old data we need to take advantage of the **Merge** command supported only by **Delta** tables (not parquet). This is an expensive but powerful command that allows us to **replace** and **add** new data in a table based on the matching of the primary keys of the new and old data table.

### Presentation Dashboard

After creating the Presentation tables we can create Dashboard from within the Databricks Notebook. In our example the tables used are the calculated_race_results.

<img src="https://github.com/dmoralis/AzureETLFormula1Project/assets/56253720/d6750208-8265-4837-a401-88034622a325"  width="48%" height="30%">
<img src="https://github.com/dmoralis/AzureETLFormula1Project/assets/56253720/7aeceb32-1317-43e2-a55d-d6c5e761ba8d"  width="48%" height="20%">





