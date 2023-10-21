# Enhanced Description of Formula 1 Azure ETL Project

## Project Overview

Welcome to the Formula 1 Azure ETL Project! This initiative is an integral part of my educational journey in Azure Databricks, facilitated through the Udemy platform. As a testament to my successful completion of the course, I hold a certificate, which can be validated at the following link: [Udemy Certificate](https://ude.my/UC-e54cd93d-2409-4334-8495-18a514c3c0d3).

The core objective of this project is to harness the power of Azure Databricks to extract Formula 1 data from the Ergast API (via manual download), meticulously transform the data into a structure suitable for insightful queries, and subsequently store it within a dedicated container for future analytical endeavors.

![Formula 1 Project Flowchart](https://github.com/dmoralis/AirflowETLWeatherProject/assets/56253720/c62296be-61a8-4df1-8d62-cb3f92a40941)
*Image source: Azure Databricks & Spark For Data Engineers (PySpark / SQL) Course by Ramesh Retnasamy*

## Technologies Utilized

In the pursuit of our Formula 1 data-driven journey, we have harnessed a potent array of technologies:

- **Azure Databricks**: The heart of our data transformation and analysis efforts.
- **PySpark - SQL**: The primary languages enabling data manipulation and querying.
- **Azure Workspace**: Our collaborative work environment.
- **Azure Resource Group**: For efficient resource management.
- **Azure Data Lake Gen2**: The repository of our data, providing scalability and reliability.
- **Azure Data Factory**: Orchestrating our ETL processes.
- **Azure Delta Lake**: Enhancing our data lake with transactional capabilities.
- **Unity Catalog**: Providing a centralized metadata repository.

## Data Insight

Our Formula 1 dataset encompasses a wealth of information, organized into various tables:

- **Circuits** (CSV)
- **Races** (CSV)
- **Constructors** (Single Line JSON)
- **Drivers** (Single Line Nested JSON)
- **Results** (Single Line JSON)
- **PitStops** (Multi Line JSON)
- **LapTimes** (Split CSV Files)
- **Qualifying** (Split Multi Line JSON Files)

Each of these tables boasts a descriptive name and serves a unique purpose. For in-depth insights into these tables, consult the official user guide: [F1DB User Guide](https://ergast.com/docs/f1db_user_guide.txt).

## Project Details

### Data Storage

Our project embraces the Azure Data Lake Gen2 throughout its lifecycle, employing three distinct phases: **Bronze**, **Silver**, and **Gold**, each residing in separate containers. Here's a breakdown of these phases:
- **Bronze**: This stage harbors the raw, unprocessed data.
- **Silver**: Data is refined and shaped here, making it ready for further analysis.
- **Gold**: The highest level of data transformation, primed for analytical applications.

### Ingestion & Transformation

**Azure Databricks** emerges as the powerhouse behind data ingestion and transformation. We employ Notebooks, dedicated Clusters, and leverage the PySpark and SQL languages to craft this transformation. The process unfolds as follows:
- Data is extracted from Formula 1 files manually uploaded to the Bronze container.
- Column names are converted from Camel Case to Snake Case, ensuring consistency.
- Processed data is stored in a defined schema or database, with the Silver container serving as its storage location.
- Production-level tables are crafted, residing in a schema with the Gold container as the base location. Key tables include:
  - **race_results**
  - **driver_standings**
  - **constructor_standings**
  - **calculated_race_results**
  
Data is incrementally added to the Data Lake Gen2 every week, categorized as both **Dimension** and **Fact** tables. Dimension tables undergo a **Full Load** due to their static nature, while Fact tables necessitate an **Incremental Load** owing to their size and frequent updates. The dimension tables comprise Circuits, Races, Constructors, and Drivers, while the fact tables consist of Results, PitStops, LapTimes, and Qualifying. Notably, to facilitate incremental loading, two additional columns, **create_date** and **update_date**, are appended to each Fact table to monitor data updates and support dynamic calculated tables.

The data update process varies as follows:
- Full Load data is overwritten every week.
- Incremental data utilizes the **Merge** command, exclusive to Delta tables (not Parquet), to replace and add new data based on matching primary keys between new and existing data.

### Configuration

Effortless access to the Storage Containers from Databricks Notebooks is made possible through a robust configuration method. We offer three distinct options for this purpose:

- **Access Key**: Create an access key from the Key Vault, safeguarded in Databricks Secrets for enhanced security. Access the containers using **abfss**.

- **Shared Access Signature (SAS)**: Ideal for short-term and low-permission access. Generate a SAS token for the container and incorporate it into Databricks Secrets. Configure your setup using these credentials.

- **Service Principal (Used in this project)**: Leverage a Service Principal with a client secret, enhancing security. Configure your setup using the **abfss** to access the desired container.

Additionally, we employ the **dbutils.fs.mount** command to streamline container access through Service Principal credentials, simplifying data retrieval without relying on abfss links.

### Dashboards

With our presentation tables in place, creating insightful dashboards becomes a seamless task. For instance, in this project, the calculated_race_results table serves as a basis for dashboard creation, offering valuable insights into Formula 1 data.

![Dashboard Image 1](https://github.com/dmoralis/AzureETLFormula1Project/assets/56253720/d6750208-8265-4837-a401-88034622a325)

![Dashboard Image 2](https://github.com/dmoralis/AzureETLFormula1Project/assets/56253720/7aeceb32-1317-43e2-a55d-d6c5e761ba8d)

### Pipeline Orchestration & Scheduling

While pipeline orchestration can be managed within the Databricks Workflow section, we opt for a more robust solution, Azure Data Factory. Our pipeline comprises two distinct sub-pipelines: Ingestion and Transformation. To ensure error-free execution, we implement conditional logic through If-statements and schedule the pipelines to run on a **Tumbling Window** trigger. This trigger activates every week on Sundays, coinciding with F1 race events, and it provides a dynamic **p_window_end_date** parameter at the close of the trigger window.

In summary, the Formula 1 Azure ETL Project is a comprehensive endeavor that seamlessly blends cutting-edge technologies, thoughtful data handling, and insightful analysis. It represents a testament to the power of Azure Databricks in transforming raw data into a valuable resource for Formula 1 enthusiasts and analysts.
