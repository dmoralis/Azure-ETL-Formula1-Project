# Formula 1 Azure ETL Project

This project is part of my course attendance about Azure Databricks, using the Udemy platform. There is a certificate available at this link [Udemy Certificate](https://ude.my/UC-e54cd93d-2409-4334-8495-18a514c3c0d3) that serves as proof of completion. The aim of this project is to extract Formula 1 data from the Ergast API (via manual download), transform the data to make it suitable for presentation queries, and finally, load it into a storage container for future use.

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

The tables in the Formula 1 dataset are as follows:

- Circuits (CSV)
- Races (CSV)
- Constructors (Single Line JSON)
- Drivers (Single Line Nested JSON)
- Results (Single Line JSON)
- PitStops (Multi Line JSON)
- LapTimes (Split CSV Files)
- Qualifying (Split Multi Line JSON Files)

Each of them has a descriptive name. For more information about these tables, you can visit the official site's user guide: [F1DB User Guide](https://ergast.com/docs/f1db_user_guide.txt)

## Project Details

### Storage
The data is saved in the same Data Lake Gen2 in all 3 phases: **Bronze**, **Silver**, **Gold** but in different containers. Bronze refers to the raw data, Silver to the processed data and Gold to the highly transformed data which is ready for analysis use. 


### Ingestion & Transformation

**Azure Databricks** is used for ingestion, supplying Notebooks, a Cluster and the languages **PySpark** and **SQL**. First we extract the F1 files that were manually uploaded to the Bronze container and convert the column names from Camel Case to Snake Case where needed. Then the resulting tables are saved in a **schema** or **database** which has as storage location the Silver container. These processed data is used to create production-level tables, which are saved in a **schema** with base location the Gold container, as managed tables:

- race_results
- driver_standings
- constructor_standings
- calculated_race_results

The data is added to the Data Lake Gen2 incrementally every week, and it's categorized as **Dimension** and **Fact** tables. Dimension tables require a **Full Load**, as they are small and don't change frequently. Fact tables require an **Incremental Load** because the full data can be large, and updates are frequent. The dimension tables include: Circuits, Races, Constructors, and Drivers, while the fact tables include: Results, PitStops, LapTimes, and Qualifying. It's important to note that due to the incremental load, two new columns are appended to each Fact table: **create_date** and **update_date** to track data updates and support dynamic calculated tables.

To perform a full load of new data every week, the old data is **overwritten**, and for incremental data, the **Merge** command is used, which is supported only by **Delta** tables, not Parquet. This command allows for the **replacement** and **addition** of new data in a table based on matching primary keys between the new and old data tables.

### Configuration

To use the Storage Containers from Databricks Notebooks, a configuration method is set up between the two services. The choices are:

- Access Key
  - Create an access key from the **Key Vault**
  - Set the access key in the Secrets scope of Databricks for security
  - Configure access using this access key
  - Access the containers using **abfss**
- Shared Access Signature (SAS) [Best for short-term and low permission accesses]
  - Right-click on the container you want to access and generate a SAS token
  - Append the SAS token to the Secrets scope of Databricks
  - Configure using these credentials
- Service Principal (Used in this project)
  - Open Microsoft Azure ID
  - Create an App and open Certificates & Secrets
  - Create a new client secret
  - Use the client-id, tenant-id, and client-secret from this App for configuration
  - Use abfss to access the desired container

Additionally, **dbutils.fs.mount** command is used to mount the whole container using Service Principal credentials, making data access more convenient without using abfss links.

### Dashboards

After creating the Presentation tables, dashboards can be created from within the Databricks Notebook. In this example, the tables used are the calculated_race_results.

<img src="https://github.com/dmoralis/AzureETLFormula1Project/assets/56253720/d6750208-8265-4837-a401-88034622a325"  width="48%" height="200px">
<img src="https://github.com/dmoralis/AzureETLFormula1Project/assets/56253720/7aeceb32-1317-43e2-a55d-d6c5e761ba8d"  width="48%" height="200px">

### Pipeline Orchestration & Schedule

Pipeline Orchestation and Schedule can take place in Databricks Workflow section, but it is prefered to use a service like Azure Data Factory because of the different capabilities it provides. 
Our pipeline consists of two sub-pipelines:
- Ingestion
- Transformation
  
Both of these pipelines use If-statements in order to avoid any errors, while waiting for a **Tumbling Window** trigger to fire every week ta Sundays, where the F1 race takes place, and provide a **p_window_end_date** parameter that is given at the end of the trigger window.




