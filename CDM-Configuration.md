# CDM Configuration for use with WebAPI

WebAPI supports configuration of one or more databases transformed to the OMOP Common Data Model (CDM) v5. In this section, we'll review the organization of the CDM and how to add the additional schemas to support operations performed by WebAPI.

## Prerequisites

This guide assumes that you have installed WebAPI which will create the necessary tables in the steps below.

## Overview

The WebAPI database created in the [WebAPI Installation Guide](WebAPI-Installation-Guide.md) allows you to configure one or more CDMs to use for performing various analyses through WebAPI. WebAPI's database contains two tables that hold this configuration: `source` and `source_daimon`. The JDBC connections to the CDM(s) are held in the `source` table and defines the connection to the CDM database. This source entry is then related to the `source_daimons` table using the `source_id` and is used to configure the schemas where the CDM, vocabulary, results and temp schemas reside. These schemas support different daimon-type functionality for WebAPI functionality. In this guide, we'll cover how to establish the additional schemas in your CDM required by WebAPI and how to configure the required `source` and `source_daimon` entries required for WebAPI to function properly.

### Schemas

The OMOP CDM contains a set of tables that are used to hold patient level data which we will refer to as the `CDM schema`. In addition, the CDM will contain tables that hold the OMOP vocabulary which we will refer to as the `Vocabulary schema`. It is most likely that the `CDM schema` and `Vocabulary schema` refer to to the same schema in the CDM database.

If you have utilized [Achilles](https://github.com/OHDSI/Achilles) to characterize your OMOP CDM, you will have created a `Results schema` to hold the results of the characterization resulting in a number of tables prefixed with `Achilles` in it. WebAPI utilizes the same `Results schema` to perform transactional operations against the CDM so that tables that require write permission can be properly partitioned away from the read-only patient data. Additionally, WebAPI also utilizes a schema for temporary storage of information when performing certain analyses. To support this work, a `temp schema` is established that enables the WebAPI to create/drop temporary tables.

The following sections will describe how to prepare your CDM for use with the OHDSI WebAPI and how to configure WebAPI to connect to the CDMs in your environment.

## Schema setup

⭐️ We would highly recommend that you run [Achilles](https://github.com/OHDSI/Achilles) to characterize your CDM data. As part of that process, you will establish a `Results schema` to hold the results. These tables are utilized by WebAPI to supplement the vocabulary search experience to include information about record counts based on CDMs configured in WebAPI. In the event that you have not done this, we'll cover how to set this up.

Ask your database administrator to establish 2 new schemas in your CDM for the `Results schema` and the `temp schema`. We'll refer to these schemas as `results` and `temp` through the remainder of this guide. Also, you will need to establish a SQL account that is used by WebAPI to access the CDM and these various schemas. For this guide, we'll refer to this account as `webapi_sa`.

To start, `webapi_sa` account requires the following permissions on these schemas:

| schema      | permissions                                 |
|-------------|---------------------------------------------|
| cdm         |  read-only                                  |
| vocabulary  |  read-only                                  |
| results     |  insert/delete/select/update                |
| temp        |  full control (create/remove tables & data) |


### Results schema setup

Once the `Results schema` is established, you will need to generate the SQL script for your CDM dialect to establish the tables that WebAPI will require to run. To do this, WebAPI provides a URL that takes parameters via the query string to generate the proper database script for your setup:

```
http://<server:port>/WebAPI/ddl/results?dialect=<your_cdm_database_dialect>&schema=<your_results_schema>&vocabSchema=<your_vocab_schema>&tempSchema=<your_temp_schema>&initConceptHierarchy=true
```

You will need to modify the URL above to point your instance of WebAPI running on `<server:port>` (default in this guide is `localhost:8080`) and then substitute the values specific to your CDM setup:

- `<your_cdm_database_dialect>`: This is one of the following: `oracle`, `postgresql`, `pdw`, `redshift`, `impala`, `netezza`, `bigquery`, or `sql server` and is based on the platform you use to host your CDM.
- `<your_results_schema>`: The schema containing your results tables
- `<your_vocab_schema>`: The schema containing your vocabulary tables
- `<your_temp_schema>`: The schema that it utilized for your temporary schema
- The `initConceptHierarchy` value in the URL is set to `true` and is used to establish the concept_hierarchy which is a cached version of the OMOP vocabulary specific to the concepts found in your CDM. This table can take a while to build and only needs to be established one time. This value can be set to `false` if you do not need to re-establish this table.

Once you have created the URL for your environment, open a browser and navigate to that URL. The resulting SQL will be displayed in the browser and your database administrator can use this script to establish the results schema.

## source and source_daimon table setup

The WebAPI `source` and `source_daimon` tables were created when you started the tomcat service with the WebAPI war deployed.  These tables must be populated with a JDBC `source` connection and corresponding `source_daimon` that specify the location for the `cdm`, `vocabulary`, `results` and `temp` schemas associated to the source in order to use the OHDSI tools. For this example it is assumed that the CDM and Vocabulary exist as a separate schema in the same database instance.  

### Example WebAPI SOURCE and SOURCE_DAIMON Inserts

```sql
INSERT INTO ohdsi.source (source_id, source_name, source_key, source_connection, source_dialect) VALUES (1, 'My Cdm', 'MY_CDM', ' jdbc:postgresql://server:5432/cdm?user={user}&password={password}', 'postgresql');

INSERT INTO webapi.source_daimon (source_daimon_id, source_id, daimon_type, table_qualifier, priority) VALUES (1,1,0, 'cdm', 0);
INSERT INTO webapi.source_daimon (source_daimon_id, source_id, daimon_type, table_qualifier, priority) VALUES (2,1,1, 'vocab', 0);
INSERT INTO webapi.source_daimon (source_daimon_id, source_id, daimon_type, table_qualifier, priority) VALUES (3,1,2, 'results', 0);
INSERT INTO webapi.source_daimon (source_daimon_id, source_id, daimon_type, table_qualifier, priority) VALUES (4,1,5, 'temp', 0);

```

The above inserts creates a source with `source_id = 1` with 4 daimon entries, one for each daimon type (0 = CDM, 1 = Vocabulary, 2 = Results, 5 = TEMP). If you'd like to configure more than 1 source for use in WebAPI, repeat the steps above and increment the `source_id` used to distinguish the sources from one another.

👉 _Note: To see the new sources, open a browser and navigate to `<server>:port/WebAPI/sources/refresh`_

## Verify Configuration
Once WebAPI is started, and the source/source_daimon inserts are complete, you should be able to open a browser to the following URL:
```
http://localhost:8080/WebAPI/source/sources
```
This should result in the following output:
```json
[{
		"sourceId": 1,
		"sourceName": "My Cdm",
		"sourceDialect": "postgresql",
		"sourceKey": "MY_CDM",
		"daimons":
		[{
				"sourceDaimonId": 1,
				"daimonType": "CDM",
				"tableQualifier": "cdm",
				"priority": "0"
			}, {
				"sourceDaimonId": 2,
				"daimonType": "Vocabulary",
				"tableQualifier": "vocab",
				"priority": "0"
			}, {
				"sourceDaimonId": 3,
				"daimonType": "Results",
				"tableQualifier": "results",
				"priority": "0"
			}, {
				"sourceDaimonId": 4,
				"daimonType": "Temp",
				"tableQualifier": "temp",
				"priority": "0"
			}
		]
	}
]
```

WebAPI is now configured and ready to serve OHDSI tools!
