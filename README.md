# Formula1 Azure ETL Project

This project is part of my course attendance about Azure Databricks, using the Udemy platform. There is a certificate in this link https://ude.my/UC-e54cd93d-2409-4334-8495-18a514c3c0d3, that is proof of completance. The aim of this project is to extract Formula 1 data from Ergast API (via manual download), transform them so they are ready for presentation queries and at last load them in a storage container for future use.

## Technologies Used

- ** Azure Databricks **
- ** PySpark - SQL **
- ** Azure Workspace **
- ** Azure Resource Group **
- ** Azure Data Lake Gen2 **
- ** Azure Data Factory **
- ** Azure Delta Lake **
- ** Unity Catalog **

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

** Azure Databricks ** is used w/ a combination of Notebooks and a Cluster in order to extract the files that was updated manually in a ** Raw ** container and convert the column names from Camel Hase to Snake Case where its needed. Then save them in a ** Processed ** container of the Data Lake Gen2, and use them to create production level tables that are saved in the ** Presentation ** container in the same data lake.

The languages that are used in the notebooks are ** PySpark ** and ** SQL **, so we can extract the data from the Raw container w/ and create from them a  ** schema ** w/ managed tables for the Processed and container. Respectively transform the data from the Processed schema and create new tables in a Presentation ** schema ** w/ managed tables:

- race_results
- driver_standings
- constructor_standings
- calculated_results

The data is coming to the data lake incrementally, that mean we need to create using ** dbutils ** 
