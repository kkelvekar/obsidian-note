Below is an updated system design note that builds on your original design and incorporates the new approach for handling detail (child) data via type-specific configuration tables and a polymorphic mapping table. You can use this as the design document for your enhanced Data Availability Dashboard.

## 1. Project Overview

**Data Availability Dashboard** is a web application integrated into the UBS Advantage Portal. It provides stakeholders with a single view of daily data load metrics (such as record counts and load dates) for multiple data domains (e.g., FxRates, Fixed Income, Security Prices, Account Master, etc.). In **Phase One**, the dashboard displays summary (master) data. In this enhanced design, when a user selects a master data row in the UI, the system will retrieve **detail data** from an external source.

Key enhancements include:

- **Master Data**: Unchanged from the original design, using the `DataDomainConfig` table (with one-to-one mapping to its source-specific configuration such as `DomainSourceGraphQL`).
- **Detail Data**: A new design that uses separate detail configuration tables (for API, Kafka, SQL, etc.) along with a polymorphic mapping table that enables many-to-one relationships. This lets multiple domains share the same detail API configuration while keeping the master schema unchanged.

---

## 2. Flow Chart Overview

1. **Frontend** (React application in UBS Advantage) calls the **Orchestrator API** (`GET /api/data-availability`) to fetch master data.
2. The **Orchestrator** reads domain configurations from the **`da_dashboard_db`** (e.g., `DataDomainConfig` and `DomainSourceGraphQL`) to get summary metrics.
3. **User Interaction**: When a user selects a specific master data row (total count data), the UI triggers a call to a new detail API endpoint.
4. The **Detail API** uses a mapping table (**DataDomainDetailSourceMapping**) to determine which external detail data configuration applies to the selected domain.
5. The system queries the appropriate detail configuration table (e.g., **ChildDataSourceApiConfig**, **ChildDataSourceKafkaConfig**, or **ChildDataSourceSqlConfig**) and calls the external service to retrieve detailed data.
6. The **Orchestrator** aggregates and returns the detail data to the UI.
7. **Security**: All API calls enforce Azure AD tokens and relevant roles (e.g., `PLATFORM-BUSINESS-ANALYST`).

---

## 3. Frontend (React – UBS Advantage Application)

- The main dashboard displays master data (total record counts and latest load dates) in a grid.
- When a user selects a master row, a detail view is launched that displays more granular data.
- The detail data is retrieved from the new detail API endpoint, which uses the mapping design described below.

---

## 4. Backend Configuration – Master Data

The master data design remains as follows:

### 4.1 DataDomainConfig (Main Table)

|Column (PK)|Type|Description|
|---|---|---|
|**Id**|`uniqueidentifier`|Unique ID for each domain config|
|**DomainName**|`nvarchar(100)`|e.g., "FxRates," "Fixed Income (FI) Analytics"|
|**SourceType**|`nvarchar(50)`|e.g., "GraphQL," "REST," "SQL," etc.|
|**IsActive**|`bit`|Indicates if the domain is active|
|**CreatedDate**|`datetime2`|For auditing|
|**UpdatedDate**|`datetime2`|For auditing|

### 4.2 DomainSourceGraphQL (Child Table for GraphQL-specific fields)

|Column (PK)|Type|Description|
|---|---|---|
|**DataDomainId**|`uniqueidentifier` FK|References `DataDomainConfig.Id`|
|**DevBaseUrl**|`nvarchar(2000)`|e.g., `https://api.cedar-dev.azpriv-cloud.ubs.net/dataservices`|
|**QaBaseUrl**|`nvarchar(2000)`|e.g., `https://api.cedar-qa.azpriv-cloud.ubs.net/dataservices`|
|**PreProdBaseUrl**|`nvarchar(2000)`|e.g., `https://api.cedar-preprod.azpriv-cloud.ubs.net/dataservices`|
|**ProdBaseUrl**|`nvarchar(2000)`|e.g., `https://api.cedar-prod.azpriv-cloud.ubs.net/dataservices`|
|**EndpointPath**|`nvarchar(500)`|Often `/<DataDomain>/graphql/` or similar|
|**Metadata**|`nvarchar(max)`|Parameters for the GraphQL query in JSON format (e.g., { "entityKeys": ["fxrate", "benchmarkholding"]})|

> _Note_: Future sources for master data (e.g., REST, SQL) will be configured in separate child tables, similar to the existing GraphQL approach.

---

## 5. Enhanced Detail Data Configuration

To support fetching detailed (child) data without modifying the master schema, we introduce two new sets of tables.

### 5.1 Type-Specific Detail Configuration Tables

These tables store configuration data for external detail data sources. Each table is dedicated to a specific technology.

#### 5.1.1 ChildDataSourceApiConfig (for API-based detail data)

```sql
CREATE TABLE [dbo].[ChildDataSourceApiConfig]
(
    [Id]                UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID(),
    [ApiName]           NVARCHAR(100)     NOT NULL,    -- e.g., 'REST_DetailAPI'
    [DevBaseUrl]        NVARCHAR(2000)    NULL,
    [QaBaseUrl]         NVARCHAR(2000)    NULL,
    [PreProdBaseUrl]    NVARCHAR(2000)    NULL,
    [ProdBaseUrl]       NVARCHAR(2000)    NULL,
    [EndpointPath]      NVARCHAR(500)     NULL,
    [Metadata]          NVARCHAR(MAX)     NULL,
    [CreatedDate]       DATETIME2         NOT NULL DEFAULT SYSUTCDATETIME(),
    [UpdatedDate]       DATETIME2         NOT NULL DEFAULT SYSUTCDATETIME(),
    
    CONSTRAINT [PK_ChildDataSourceApiConfig] PRIMARY KEY ([Id])
);
```

#### 5.1.2 ChildDataSourceKafkaConfig (for Kafka-based detail data)

```sql
CREATE TABLE [dbo].[ChildDataSourceKafkaConfig]
(
    [Id]                 UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID(),
    [TopicName]          NVARCHAR(200)     NOT NULL,    -- e.g., 'DetailDataTopic'
    [BootstrapServers]   NVARCHAR(1000)    NOT NULL,
    [ConsumerGroup]      NVARCHAR(200)     NULL,
    [Metadata]           NVARCHAR(MAX)     NULL,
    [CreatedDate]        DATETIME2         NOT NULL DEFAULT SYSUTCDATETIME(),
    [UpdatedDate]        DATETIME2         NOT NULL DEFAULT SYSUTCDATETIME(),
    
    CONSTRAINT [PK_ChildDataSourceKafkaConfig] PRIMARY KEY ([Id])
);
```

#### 5.1.3 ChildDataSourceSqlConfig (for SQL-based detail data)

```sql
CREATE TABLE [dbo].[ChildDataSourceSqlConfig]
(
    [Id]                UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID(),
    [ConnectionString]  NVARCHAR(2000)    NOT NULL,
    [Query]             NVARCHAR(MAX)     NOT NULL,    -- The SQL query to fetch detail data
    [Metadata]          NVARCHAR(MAX)     NULL,
    [CreatedDate]       DATETIME2         NOT NULL DEFAULT SYSUTCDATETIME(),
    [UpdatedDate]       DATETIME2         NOT NULL DEFAULT SYSUTCDATETIME(),
    
    CONSTRAINT [PK_ChildDataSourceSqlConfig] PRIMARY KEY ([Id])
);
```

### 5.2 Polymorphic Mapping Table for Detail Data

This table creates a many-to-one mapping between master domains and detail data source configurations. It enables multiple domains to share the same detail data source record regardless of the underlying technology.

```sql
CREATE TABLE [dbo].[DataDomainDetailSourceMapping]
(
    [Id]                      UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID(),
    [DataDomainConfigId]      UNIQUEIDENTIFIER NOT NULL, -- References DataDomainConfig.Id
    [DataSourceType]          NVARCHAR(50)      NOT NULL, -- E.g., 'API', 'Kafka', 'SQL'
    [DataSourceConfigId]      UNIQUEIDENTIFIER  NOT NULL, -- The corresponding configuration record's Id
    [CreatedDate]             DATETIME2         NOT NULL DEFAULT SYSUTCDATETIME(),
    [UpdatedDate]             DATETIME2         NOT NULL DEFAULT SYSUTCDATETIME(),
    
    CONSTRAINT [PK_DataDomainDetailSourceMapping] PRIMARY KEY ([Id]),
    CONSTRAINT [FK_DomainDetailMapping_DataDomainConfig] 
         FOREIGN KEY ([DataDomainConfigId])
         REFERENCES [dbo].[DataDomainConfig]([Id])
);
```

> **Important**: The column **DataSourceConfigId** is polymorphic; its value will refer to a row in one of the type-specific detail tables. The **DataSourceType** discriminator (e.g., 'API', 'Kafka', 'SQL') tells the application which table to query.

---

## 6. Domain Classes & Architecture

### 6.1 Domain Layer

The core domain entities remain unchanged:

#### DataDomain

```csharp
public class DataDomain
{
    public string Name { get; set; }  
    public List<DataMetric> Metrics { get; set; }
}
```

#### DataMetric

```csharp
public class DataMetric
{
    public int Count { get; set; }  
    public DateTime Date { get; set; }
}
```

### 6.2 Application Layer

- **Master Data**:  
    The orchestrator fetches domain configurations from **DataDomainConfig** and calls the appropriate source service (e.g., GraphQL via **DomainSourceGraphQL**) to obtain summary metrics.
    
- **Detail Data**:  
    When a user selects a master row, the orchestrator uses the **DataDomainDetailSourceMapping** table to resolve:
    
    1. The **DataSourceType**.
    2. The associated configuration record (by querying the correct detail config table based on the type).
    
    The appropriate detail service (API, Kafka consumer, or SQL query executor) is then invoked to retrieve the detailed data.
    

---

## 7. Security & Deployment

### 7.1 Security

- **User Access**:  
    Authentication and role-based access (e.g., `PLATFORM-BUSINESS-ANALYST`) are enforced via Azure AD tokens in both master and detail API calls.
    
- **End-to-End Security**:  
    Downstream services (GraphQL, REST APIs, Kafka topics, SQL databases) also enforce appropriate security measures, such as token-based authentication or connection string encryption.
    

### 7.2 Deployment

- **Orchestrator (Backend)**:  
    Deployed as a containerized .NET 8 application on UBS ADV AKS.
    
- **Frontend (UBS Neo)**:  
    Deployed using existing procedures via GitLab CI/CD.
    
- **Database (`da_dashboard_db`)**:  
    Managed using Flyway migrations or an equivalent tool to handle schema updates for both master and detail configuration tables.
    

---

## 8. Conclusion

This enhanced design builds upon the existing Data Availability Dashboard system by preserving the master data configuration while introducing a flexible, type-specific solution for detail data retrieval. By using separate configuration tables for each detail source type and a polymorphic mapping table (**DataDomainDetailSourceMapping**), the solution can support many-to-one relationships without changing the master schema. This design ensures modularity, extensibility, and clean separation of concerns as new data source types are introduced.

---
