---
title: T-SQL feature in Synapse SQL pool
description: List of Transact-SQL features that are available in Synapse SQL.
services: synapse analytics
author: jovanpop-msft
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: jovanpop
ms.reviewer: jrasnick
---

# Transact-SQL features supported in Azure Synapse SQL

Azure Synapse SQL is a big data analytic service that enables you to query and analyze your data using the T-SQL language. You can use standard ANSI-compliant dialect of SQL language used on SQL Server and Azure SQL Database for data analysis. 

Transact-SQL language is used in serverless SQL pool and dedicated model can reference different objects and has some differences in the set of supported features. In this page, you can find high-level Transact-SQL language differences between consumption models of Synapse SQL.

## Database objects

Consumption models in Synapse SQL enable you to use different database objects. The comparison of supported object types is shown in the following table:

|   | Dedicated | Serverless |
| --- | --- | --- |
| **Tables** | [Yes](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?view=azure-sqldw-latest&preserve-view=true) | No, the in-database tables are not supported. Serverless SQL pool can query only [external tables](develop-tables-external-tables.md?tabs=native) that reference data placed on [Azure Storage](#storage-options) |
| **Views** | [Yes](/sql/t-sql/statements/create-view-transact-sql?view=azure-sqldw-latest&preserve-view=true). Views can use [query language elements](#query-language) that are available in dedicated model. | [Yes](/sql/t-sql/statements/create-view-transact-sql?view=azure-sqldw-latest&preserve-view=true), you can create views over [external tables](develop-tables-external-tables.md?tabs=native) and other views. Views can use [query language elements](#query-language) that are available in serverless model. |
| **Schemas** | [Yes](/sql/t-sql/statements/create-schema-transact-sql?view=azure-sqldw-latest&preserve-view=true) | [Yes](/sql/t-sql/statements/create-schema-transact-sql?view=azure-sqldw-latest&preserve-view=true), schemas are supported. |
| **Temporary tables** | [Yes](../sql-data-warehouse/sql-data-warehouse-tables-temporary.md?context=/azure/synapse-analytics/context/context) | No, temporary tables might be used just to store some information from system views. |
| **User defined procedures** | [Yes](/sql/t-sql/statements/create-procedure-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Yes, stored procedures can be placed in any user databases (not `master` database). |
| **User defined functions** | [Yes](/sql/t-sql/statements/create-function-sql-data-warehouse?view=azure-sqldw-latest&preserve-view=true) | Yes, only inline table-valued functions. Scalar user defined functions are not supported. |
| **Triggers** | No | No, serverless SQL pools do not allow changing data, so the triggers cannot react on data changes. |
| **External tables** | [Yes](/sql/t-sql/statements/create-external-table-transact-sql?view=azure-sqldw-latest&preserve-view=true). See supported [data formats](#data-formats). | [Yes](/sql/t-sql/statements/create-external-table-transact-sql?view=azure-sqldw-latest&preserve-view=true). See the supported [data formats](#data-formats). |
| **Caching queries** | Yes, multiple forms (SSD-based caching, in-memory, resultset caching). In addition, Materialized View are supported | No. Only file statistics are cached. |
| **Table variables** | [No](/sql/t-sql/data-types/table-transact-sql?view=azure-sqldw-latest&preserve-view=true), use temporary tables | No, table variables are not supported. |
| **[Table distribution](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?context=/azure/synapse-analytics/context/context)**               | Yes | No, table distributions are not supported. |
| **[Table indexes](../sql-data-warehouse/sql-data-warehouse-tables-index.md?context=/azure/synapse-analytics/context/context)**                           | Yes | No, indexes are not supported. |
| **[Table partitions](../sql-data-warehouse/sql-data-warehouse-tables-partition.md?context=/azure/synapse-analytics/context/context)**                     | Yes | No, only external tables that are synchronized from the Apache Spark pools can be partitioned per folders. |
| **[Statistics](develop-tables-statistics.md)**            | Yes | Yes |
| **Workload management, resource classes, and concurrency control** | Yes, see [workload management, resource classes, and concurrency control](../sql-data-warehouse/resource-classes-for-workload-management.md?context=/azure/synapse-analytics/context/context).    | No, serverless SQL pool automatically manages the resources. |
| **Cost control** | Yes, using scale-up and scale-down actions. | Yes, using [the Azure portal or T-SQL procedure](./data-processed.md#cost-control). |

## Query language

Query languages used in Synapse SQL can have different supported features depending on consumption model. The following table outlines the most important query language differences in Transact-SQL dialects:

|   | Dedicated | Serverless |
| --- | --- | --- |
| **SELECT statement** | Yes. Transact-SQL query clauses [FOR XML/FOR JSON](/sql/t-sql/queries/select-for-clause-transact-sql?view=azure-sqldw-latest&preserve-view=true), [MATCH](/sql/t-sql/queries/match-sql-graph?view=azure-sqldw-latest&preserve-view=true), OFFSET/FETCH  are not supported. | Yes. Transact-SQL query clauses [FOR XML](/sql/t-sql/queries/select-for-clause-transact-sql?view=azure-sqldw-latest&preserve-view=true), [MATCH](/sql/t-sql/queries/match-sql-graph?view=azure-sqldw-latest&preserve-view=true), [PREDICT](/sql/t-sql/queries/predict-transact-sql?view=azure-sqldw-latest&preserve-view=true), GROUPNG SETS, and query hints are not supported. |
| **INSERT statement** | Yes | No, upload new data to Data lake using other tools. |
| **UPDATE statement** | Yes | No, but data updated using Spark is automatically available in serverless pool. |
| **DELETE statement** | Yes | No, but data deleted using Spark is automatically available in serverless pool. |
| **MERGE statement** | Yes ([preview](/sql/t-sql/statements/merge-transact-sql?view=azure-sqldw-latest&preserve-view=true)) | No, but data merged using Spark is automatically available in serverless pool. |
| **[Transactions](develop-transactions.md)** | Yes | Yes, applicable on meta-data objects. |
| **[Labels](develop-label.md)** | Yes | No |
| **Data load** | Yes. Preferred utility is [COPY](/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest&preserve-view=true) statement, but the system supports both BULK load (BCP) and [CETAS](/sql/t-sql/statements/create-external-table-as-select-transact-sql?view=azure-sqldw-latest&preserve-view=true) for data loading. | No, you can initially load data into an external table using CETAS statement. |
| **Data export** | Yes. Using [CETAS](/sql/t-sql/statements/create-external-table-as-select-transact-sql?view=azure-sqldw-latest&preserve-view=true). | Yes. Using [CETAS](/sql/t-sql/statements/create-external-table-as-select-transact-sql?view=azure-sqldw-latest&preserve-view=true). |
| **Types** | Yes, all Transact-SQL types except [cursor](/sql/t-sql/data-types/cursor-transact-sql?view=azure-sqldw-latest&preserve-view=true), [hierarchyid](/sql/t-sql/data-types/hierarchyid-data-type-method-reference?view=azure-sqldw-latest&preserve-view=true), [ntext, text, and image](/sql/t-sql/data-types/ntext-text-and-image-transact-sql?view=azure-sqldw-latest&preserve-view=true), [rowversion](/sql/t-sql/data-types/rowversion-transact-sql?view=azure-sqldw-latest&preserve-view=true), [Spatial Types](/sql/t-sql/spatial-geometry/spatial-types-geometry-transact-sql?view=azure-sqldw-latest&preserve-view=true), [sql\_variant](/sql/t-sql/data-types/sql-variant-transact-sql?view=azure-sqldw-latest&preserve-view=true), and [xml](/sql/t-sql/xml/xml-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Yes, all Transact-SQL types except [cursor](/sql/t-sql/data-types/cursor-transact-sql?view=azure-sqldw-latest&preserve-view=true), [hierarchyid](/sql/t-sql/data-types/hierarchyid-data-type-method-reference?view=azure-sqldw-latest&preserve-view=true), [ntext, text, and image](/sql/t-sql/data-types/ntext-text-and-image-transact-sql?view=azure-sqldw-latest&preserve-view=true), [rowversion](/sql/t-sql/data-types/rowversion-transact-sql?view=azure-sqldw-latest&preserve-view=true), [Spatial Types](/sql/t-sql/spatial-geometry/spatial-types-geometry-transact-sql?view=azure-sqldw-latest&preserve-view=true), [sql\_variant](/sql/t-sql/data-types/sql-variant-transact-sql?view=azure-sqldw-latest&preserve-view=true), [xml](/sql/t-sql/xml/xml-transact-sql?view=azure-sqldw-latest&preserve-view=true), and Table type |
| **Cross-database queries** | No | Yes, including [USE](/sql/t-sql/language-elements/use-transact-sql?view=azure-sqldw-latest&preserve-view=true) statement. |
| **Built-in/system functions (analysis)** | Yes, all Transact-SQL [Analytic](/sql/t-sql/functions/analytic-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), Conversion, [Date and Time](/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), Logical, [Mathematical](/sql/t-sql/functions/mathematical-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true) functions, except [CHOOSE](/sql/t-sql/functions/logical-functions-choose-transact-sql?view=azure-sqldw-latest&preserve-view=true) and [PARSE](/sql/t-sql/functions/parse-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Yes, all Transact-SQL [Analytic](/sql/t-sql/functions/analytic-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), Conversion, [Date and Time](/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), Logical, [Mathematical](/sql/t-sql/functions/mathematical-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true) functions. |
| **Built-in/system functions ([string](/sql/t-sql/functions/string-functions-transact-sql))** | Yes. All Transact-SQL [String](/sql/t-sql/functions/string-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), [JSON](/sql/t-sql/functions/json-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), and Collation functions, except [STRING_ESCAPE](/sql/t-sql/functions/string-escape-transact-sql?view=azure-sqldw-latest&preserve-view=true) and [TRANSLATE](/sql/t-sql/functions/translate-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Yes. All Transact-SQL [String](/sql/t-sql/functions/string-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), [JSON](/sql/t-sql/functions/json-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), and Collation functions. |
| **Built-in/system functions ([Cryptographic](/sql/t-sql/functions/cryptographic-functions-transact-sql))** | Some | No |
| **Built-in/system table-value functions** | Yes, [Transact-SQL Rowset functions](/sql/t-sql/functions/functions?view=azure-sqldw-latest&preserve-view=true#rowset-functions), except [OPENXML](/sql/t-sql/functions/openxml-transact-sql?view=azure-sqldw-latest&preserve-view=true), [OPENDATASOURCE](/sql/t-sql/functions/opendatasource-transact-sql?view=azure-sqldw-latest&preserve-view=true), [OPENQUERY](/sql/t-sql/functions/openquery-transact-sql?view=azure-sqldw-latest&preserve-view=true), and [OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Yes, [Transact-SQL Rowset functions](/sql/t-sql/functions/functions?view=azure-sqldw-latest&preserve-view=true#rowset-functions), except [OPENXML](/sql/t-sql/functions/openxml-transact-sql?view=azure-sqldw-latest&preserve-view=true), [OPENDATASOURCE](/sql/t-sql/functions/opendatasource-transact-sql?view=azure-sqldw-latest&preserve-view=true), and [OPENQUERY](/sql/t-sql/functions/openquery-transact-sql?view=azure-sqldw-latest&preserve-view=true)  |
| **Built-in/system aggregates** |  Transact-SQL built-in aggregates except, except [CHECKSUM_AGG](/sql/t-sql/functions/checksum-agg-transact-sql?view=azure-sqldw-latest&preserve-view=true) and [GROUPING_ID](/sql/t-sql/functions/grouping-id-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Transact-SQL built-in aggregates. |
| **Operators** | Yes, all [Transact-SQL operators](/sql/t-sql/language-elements/operators-transact-sql?view=azure-sqldw-latest&preserve-view=true) except [!>](/sql/t-sql/language-elements/not-greater-than-transact-sql?view=azure-sqldw-latest&preserve-view=true) and [!<](/sql/t-sql/language-elements/not-less-than-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Yes, all [Transact-SQL operators](/sql/t-sql/language-elements/operators-transact-sql?view=azure-sqldw-latest&preserve-view=true)  |
| **Control of flow** | Yes. All [Transact-SQL Control-of-flow statement](/sql/t-sql/language-elements/control-of-flow?view=azure-sqldw-latest&preserve-view=true) except [CONTINUE](/sql/t-sql/language-elements/continue-transact-sql?view=azure-sqldw-latest&preserve-view=true), [GOTO](/sql/t-sql/language-elements/goto-transact-sql?view=azure-sqldw-latest&preserve-view=true), [RETURN](/sql/t-sql/language-elements/return-transact-sql?view=azure-sqldw-latest&preserve-view=true), [USE](/sql/t-sql/language-elements/use-transact-sql?view=azure-sqldw-latest&preserve-view=true), and [WAITFOR](/sql/t-sql/language-elements/waitfor-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Yes. All [Transact-SQL Control-of-flow statement](/sql/t-sql/language-elements/control-of-flow?view=azure-sqldw-latest&preserve-view=true) SELECT query in `WHILE (...)` condition |
| **DDL statements (CREATE, ALTER, DROP)** | Yes. All Transact-SQL DDL statement applicable to the supported object types | Yes. All Transact-SQL DDL statement applicable to the supported object types |

## Security

Synapse SQL pools enable you to use built-in security features to secure your data and control access. The following table compares high-level differences between Synapse SQL consumption models.

|   | Dedicated | Serverless |
| --- | --- | --- |
| **Logins** | N/A (only contained users are supported in databases) | Yes |
| **Users** |  N/A (only contained users are supported in databases) | Yes |
| **[Contained users](/sql/relational-databases/security/contained-database-users-making-your-database-portable?view=azure-sqldw-latest&preserve-view=true)** | Yes. **Note:** only one Azure AD user can be unrestricted admin | No |
| **SQL username/password authentication**| Yes | Yes |
| **Azure Active Directory (Azure AD) authentication**| Yes, Azure AD users | Yes, Azure AD logins and users |
| **Storage Azure Active Directory (Azure AD) passthrough authentication** | Yes | Yes |
| **Storage SAS token authentication** | No | Yes, using [DATABASE SCOPED CREDENTIAL](/sql/t-sql/statements/create-database-scoped-credential-transact-sql?view=azure-sqldw-latest&preserve-view=true) in [EXTERNAL DATA SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql?view=azure-sqldw-latest&preserve-view=true) or instance-level [CREDENTIAL](/sql/t-sql/statements/create-credential-transact-sql?view=azure-sqldw-latest&preserve-view=true). |
| **Storage Access Key authentication** | Yes, using [DATABASE SCOPED CREDENTIAL](/sql/t-sql/statements/create-database-scoped-credential-transact-sql?view=azure-sqldw-latest&preserve-view=true) in [EXTERNAL DATA SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql?view=azure-sqldw-latest&preserve-view=true) | No |
| **Storage [Managed Identity](../../data-factory/data-factory-service-identity.md?context=/azure/synapse-analytics/context/context&tabs=synapse-analytics) authentication** | Yes, using [Managed Service Identity Credential](../../azure-sql/database/vnet-service-endpoint-rule-overview.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&preserve-view=true&toc=%2fazure%2fsynapse-analytics%2ftoc.json&view=azure-sqldw-latest&preserve-view=true) | Yes, using `Managed Identity` credential. |
| **Storage Application identity authentication** | [Yes](/sql/t-sql/statements/create-external-data-source-transact-sql?view=azure-sqldw-latest&preserve-view=true) | No |
| **Server-level roles** | No | Yes, sysadmin, public, and other server-roles are supported |
| **SERVER SCOPED CREDENTIAL** | No | Yes, the server scoped credentials are used by the `OPENROWSET` function that do not uses explicit data source. |
| **Permissions - [Server-level](/sql/relational-databases/security/authentication-access/server-level-roles)** | No | Yes, for example, `CONNECT ANY DATABASE` and `SELECT ALL USER SECURABLES` enable a user to read data from any databases. |
| **Database-scoped roles** | Yes | Yes |
| **DATABASE SCOPED CREDENTIAL** | Yes, used in external data sources. | Yes, used in external data sources. |
| **Permissions - [Database-level](/sql/relational-databases/security/authentication-access/database-level-roles?view=azure-sqldw-latest&preserve-view=true)** | Yes | Yes |
| **Permissions - Schema-level** | Yes, including ability to GRANT, DENY, and REVOKE permissions to users/logins on the schema | Yes, including ability to GRANT, DENY, and REVOKE permissions to users/logins on the schema |
| **Permissions - Object-level** | Yes, including ability to GRANT, DENY, and REVOKE permissions to users | Yes, including ability to GRANT, DENY, and REVOKE permissions to users/logins on the system objects that are supported |
| **Permissions - [Column-level security](../sql-data-warehouse/column-level-security.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json)** | Yes | Yes |
| **Built-in/system security &amp; identity functions** | Some Transact-SQL security functions and operators:  `CURRENT_USER`, `HAS_DBACCESS`, `IS_MEMBER`, `IS_ROLEMEMBER`, `SESSION_USER`, `SUSER_NAME`, `SUSER_SNAME`, `SYSTEM_USER`, `USER`, `USER_NAME`, `EXECUTE AS`, `OPEN/CLOSE MASTER KEY` | Some Transact-SQL security functions and operators:  `CURRENT_USER`, `HAS_DBACCESS`, `HAS_PERMS_BY_NAME`, `IS_MEMBER', 'IS_ROLEMEMBER`, `IS_SRVROLEMEMBER`, `SESSION_USER`, `SESSION_CONTEXT`, `SUSER_NAME`, `SUSER_SNAME`, `SYSTEM_USER`, `USER`, `USER_NAME`, `EXECUTE AS`, and `REVERT`. Security functions cannot be used to query external data (store the result in variable that can be used in the query).  |
| **Row-level security** | [Yes](/sql/relational-databases/security/row-level-security?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | No built-in support. Use custom views as a [workaround](https://techcommunity.microsoft.com/t5/azure-synapse-analytics-blog/how-to-implement-row-level-security-in-serverless-sql-pools/ba-p/2354759). |
| **Transparent Data Encryption (TDE)** | [Yes](../../azure-sql/database/transparent-data-encryption-tde-overview.md) | No | 
| **Data Discovery & Classification** | [Yes](../../azure-sql/database/data-discovery-and-classification-overview.md) | No |
| **Vulnerability Assessment** | [Yes](../../azure-sql/database/sql-vulnerability-assessment.md) | No |
| **Advanced Threat Protection** | [Yes](../../azure-sql/database/threat-detection-overview.md)
| **Auditing** | [Yes](../../azure-sql/database/auditing-overview.md) | Yes |
| **[Firewall rules](../security/synapse-workspace-ip-firewall.md)**| Yes | Yes |
| **[Private endpoint](../security/synapse-workspace-managed-private-endpoints.md)**| Yes | Yes |

Dedicated SQL pool and serverless SQL pool use standard Transact-SQL language to query data. For detailed differences, look at the [Transact-SQL language reference](/sql/t-sql/language-reference).

## Tools

You can use various tools to connect to Synapse SQL to query data.

|   | Dedicated | Serverless |
| --- | --- | --- |
| **Synapse Studio** | Yes, SQL scripts | Yes, SQL scripts |
| **Power BI** | Yes | [Yes](tutorial-connect-power-bi-desktop.md) |
| **Azure Analysis Service** | Yes | Yes |
| **Azure Data Studio** | Yes | Yes, version 1.18.0 or higher. SQL scripts and SQL Notebooks are supported. |
| **SQL Server Management Studio** | Yes | Yes, version 18.5 or higher |

> [!NOTE]
> You can use SSMS to connect to serverless SQL pool and query. It is partially supported starting from version 18.5, you can use it to connect and query only.

Most of the applications use standard Transact-SQL language can query both dedicated and serverless consumption models of Synapse SQL.

## Storage options

Data that is analyzed can be stored on various storage types. The following table lists all available storage options:

|   | Dedicated | Serverless |
| --- | --- | --- |
| **Internal storage** | Yes | No, data is placed in Azure Data Lake or cosmos DB analytical storage. |
| **Azure Data Lake v2** | Yes | Yes, you can use external tables and the `OPENROWSET` function to read data from ADLS. |
| **Azure Blob Storage** | Yes | Yes, you can use external tables and the `OPENROWSET` function to read data from Azure Blob Storage. |
| **Azure SQL (remote)** | No | No, serverless SQL pool cannot reference Azure SQL database. You can reference serverless SQL pools from Azure SQL using [elastic queries](https://devblogs.microsoft.com/azure-sql/read-azure-storage-files-using-synapse-sql-external-tables/) or [linked servers](https://devblogs.microsoft.com/azure-sql/linked-server-to-synapse-sql-to-implement-polybase-like-scenarios-in-managed-instance) |
| **Dataverse** | No | Yes, using [Synapse link](https://docs.microsoft.com/powerapps/maker/data-platform/azure-synapse-link-data-lake). |
| **Azure CosmosDB transactional storage** | No | No, use Spark pools to update the Cosmos DB transactional storage. |
| **Azure CosmosDB analytical storage** | No | Yes, using [Synapse Link](../../cosmos-db/synapse-link.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json) |
| **Apache Spark tables (in workspace)** | No | Only PARQUET and CSV tables using [metadata synchronization](develop-storage-files-spark-tables.md) |
| **Apache Spark tables (remote)** | No | No |
| **Databricks tables (remote)** | No | No |

## Data formats

Data that is analyzed can be stored in various storage formats. The following table lists all available data formats that can be analyzed:

|   | Dedicated | Serverless |
| --- | --- | --- |
| **Delimited** | [Yes](/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest&preserve-view=true) | [Yes](query-single-csv-file.md) |
| **CSV** | Yes (multi-character delimiters not supported) | [Yes](query-single-csv-file.md) |
| **Parquet** | [Yes](/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest&preserve-view=true) | [Yes](query-parquet-files.md), including files with [nested types](query-parquet-nested-types.md) |
| **Hive ORC** | [Yes](/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest&preserve-view=true) | No |
| **Hive RC** | [Yes](/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest&preserve-view=true) | No |
| **JSON** | Yes | [Yes](query-json-files.md) |
| **Avro** | No | No |
| **[Delta Lake](https://delta.io/)** | No | [Yes](query-delta-lake-format.md) |
| **[CDM](/common-data-model/)** | No | No |

## Next steps
Additional information on best practices for dedicated SQL pool and serverless SQL pool can be found in the following articles:

- [Best Practices for dedicated SQL pool](best-practices-dedicated-sql-pool.md)
- [Best practices for serverless SQL pool](best-practices-serverless-sql-pool.md)
