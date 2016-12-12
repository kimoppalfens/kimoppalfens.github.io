---
title: "SCCM Telemetry data unhashed – revisited for Security Auditors"
author: Kim Oppalfens
related: true
date: 2016-03-18
categories:
  - SCCM
  - Configmgr
tags:
  - Telemetry
---

A while ago, I wrote a pretty extensive blog on telemetry, 

How it worked,

What data was collected,

And most importantly how to see what the hashed data that is uploaded actually means.

You can read that blog here: [http://www.oscc.be/sccm/configuration-manager-telemetry-usage-metrics-work-in-progress-(draft)/](http://www.oscc.be/sccm/configuration-manager-telemetry-usage-metrics-work-in-progress-(draft)/)

I ended that blog on an open note, and with a couple of stored procedures that generated telemetry data containing hashed data that still needed SQL queries to see the unhashed data.

This blog post is an attempt to list all the stored procedures that contain hashed data, and the corresponding modified SQL statement to see the data unhashed.

The stored procedures that contain hashed data are:

 
## TEL_Content_DPState ##  

### Original Query ###


```sql
SELECT dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), [ContentID]), 'SHA256') AS [ContentID]
	,[State]
	,COUNT([State]) AS [StateCount]
FROM [ContentDPMap] WITH (NOLOCK)
WHERE [AccessType] = 1
GROUP BY [ContentID]
	,[State]
ORDER BY [ContentID] ASC ,[State] ASC
``` 

### Query Including the Unhashed data alongside the hashed data (ContentID Field is hashed)

```sql
SELECT dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), [ContentID]), 'SHA256') AS [HashedContentID]
	,ContentID
	,[State]
	,COUNT([State]) AS [StateCount]
FROM [ContentDPMap] WITH (NOLOCK)
WHERE [AccessType] = 1
GROUP BY [ContentID]
	,[State]
ORDER BY [ContentID] ASC
	,[State] ASC
```
 

## TEL_Content_Package ##

### Original Query ###


```sql
SELECT dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), [PkgID]), 'SHA256') AS [PkgID]
	,[SourceSize]
	,[ShareType]
	,[PackageType]
	,[PkgFlags]
	,[LastRefresh]
	,[SourceVersion]
	,(
		CASE 
			WHEN [AlternateContentProviders] = ''
				OR [AlternateContentProviders] IS NULL
				THEN 0
			ELSE 1
			END
		) AS [AlternateContentProviders]
FROM [SMSPackages_G] WITH (NOLOCK)
```

### Query Including the Unhashed data alongside the hashed data

```sql
SELECT dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), [PkgID]), 'SHA256') AS [HashedPkgID]
	,PkgID
	,[SourceSize]
	,[ShareType]
	,[PackageType]
	,[PkgFlags]
	,[LastRefresh]
	,[SourceVersion]
	,(
		CASE 
			WHEN [AlternateContentProviders] = ''
				OR [AlternateContentProviders] IS NULL
				THEN 0
			ELSE 1
			END
		) AS [AlternateContentProviders]
FROM [SMSPackages_G] WITH (NOLOCK)
```


## TEL_DCM_BuiltinSettings

### Original Query

;WITH RefSettingList AS

(

    SELECT st.Setting_UniqueID, dcm_ci.CI_ID                 

    FROM v_CISettings st 

    JOIN CI_SettingReferences ref on ref.Setting_ID = st.Setting_ID 

    JOIN CI_Rules rl on rl.Rule_ID = ref.Rule_ID         

    JOIN CI_ConfigurationItems settings_ci on settings_ci.CI_ID = st.CI_ID 

    JOIN CI_ConfigurationItems dcm_ci on dcm_ci.CI_ID = rl.CI_ID 

    WHERE dcm_ci.IsLatest = 1 and dcm_ci.IsTombstoned = 0 and dcm_ci.IsExpired = 0 /* Limit to latest CI references */

     and settings_ci.IsLatest = 1 and settings_ci.IsTombstoned = 0 and settings_ci.IsExpired = 0 

     and settings_ci.IsUserDefined = 0 /* Limit to built-in setting definition CIs */

),

DeployedSettingList AS

(

    \--Cover direct CI deployments 

    SELECT sl.Setting_UniqueID, sl.CI_ID 

    FROM RefSettingList sl 

    JOIN CI_AssignmentTargetedCIs cit on cit.CI_ID = sl.CI_ID     

    JOIN CI_CIAssignments cia on cia.AssignmentID = cit.AssignmentID 

    WHERE cia.IsTombstoned = 0 

     

    UNION

     

    \--Cover CI deployments via Baseline 

    SELECT sl.Setting_UniqueID, sl.CI_ID 

    FROM RefSettingList sl 

    JOIN CI_ConfigurationItemRelations cir on cir.ToCI_ID = sl.CI_ID 

    JOIN CI_ConfigurationItems baseline on baseline.CI_ID = cir.FromCI_ID     

    JOIN CI_AssignmentTargetedCIs cit on cit.CI_ID = baseline.CI_ID     

    JOIN CI_CIAssignments cia on cia.AssignmentID = cit.AssignmentID 

    WHERE cia.IsTombstoned = 0 

     and baseline.CIType_ID = 2 and baseline.IsLatest = 1 and baseline.IsExpired = 0 

     and baseline.IsTombstoned = 0 and baseline.IsEnabled = 1 

)     

 

\--Set upper bound in case built-in set expands rapidly 

SELECT TOP 1000 

    dbo.fnConvertBinaryToBase64String(

        dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), st.Setting_UniqueID), 'SHA256')) AS Setting_UniqueID,             

    COUNT(distinct st.CI_ID) AS CountDeployedCIs 

    FROM DeployedSettingList st     

    GROUP BY st.Setting_UniqueID 

    ORDER BY CountDeployedCIs DESC

### Query Including the Unhashed data alongside the hashed data

;WITH RefSettingList AS

(

    SELECT st.Setting_UniqueID, dcm_ci.CI_ID                 

    FROM v_CISettings st 

    JOIN CI_SettingReferences ref on ref.Setting_ID = st.Setting_ID 

    JOIN CI_Rules rl on rl.Rule_ID = ref.Rule_ID         

    JOIN CI_ConfigurationItems settings_ci on settings_ci.CI_ID = st.CI_ID 

    JOIN CI_ConfigurationItems dcm_ci on dcm_ci.CI_ID = rl.CI_ID 

    WHERE dcm_ci.IsLatest = 1 and dcm_ci.IsTombstoned = 0 and dcm_ci.IsExpired = 0 /* Limit to latest CI references */

     and settings_ci.IsLatest = 1 and settings_ci.IsTombstoned = 0 and settings_ci.IsExpired = 0 

     and settings_ci.IsUserDefined = 0 /* Limit to built-in setting definition CIs */

),

DeployedSettingList AS

(

    \--Cover direct CI deployments 

    SELECT sl.Setting_UniqueID, sl.CI_ID 

    FROM RefSettingList sl 

    JOIN CI_AssignmentTargetedCIs cit on cit.CI_ID = sl.CI_ID     

    JOIN CI_CIAssignments cia on cia.AssignmentID = cit.AssignmentID 

    WHERE cia.IsTombstoned = 0 

     

    UNION

     

    \--Cover CI deployments via Baseline 

    SELECT sl.Setting_UniqueID, sl.CI_ID 

    FROM RefSettingList sl 

    JOIN CI_ConfigurationItemRelations cir on cir.ToCI_ID = sl.CI_ID 

    JOIN CI_ConfigurationItems baseline on baseline.CI_ID = cir.FromCI_ID     

    JOIN CI_AssignmentTargetedCIs cit on cit.CI_ID = baseline.CI_ID     

    JOIN CI_CIAssignments cia on cia.AssignmentID = cit.AssignmentID 

    WHERE cia.IsTombstoned = 0 

     and baseline.CIType_ID = 2 and baseline.IsLatest = 1 and baseline.IsExpired = 0 

     and baseline.IsTombstoned = 0 and baseline.IsEnabled = 1 

)     

 

\--Set upper bound in case built-in set expands rapidly 

SELECT TOP 1000 

    dbo.fnConvertBinaryToBase64String(

        dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), st.Setting_UniqueID), 'SHA256')) AS HashedSetting_UniqueID,st.Setting_UniqueID,

    COUNT(distinct st.CI_ID) AS CountDeployedCIs 

    FROM DeployedSettingList st     

    GROUP BY st.Setting_UniqueID 

    ORDER BY CountDeployedCIs DESC

## TEL_EAS_Connectors

### Original Query

DECLARE @ConnectorId TABLE

(

    ConnectorID nvarchar(40) not null

)

 

DECLARE @ExchangeConnectorTable TABLE

(

    ConnectorID nvarchar(40) not null,

    ConfiguredExchangeServer nvarchar(256) not null,

    IsHosted int not null

)

 

DECLARE @ExchangePolicyTable TABLE

(

    ConnectorID nvarchar(40) not null,

    IsPolicyDeployed bit not null

 

)

 

\-- Get unique IDs for exchange connectors     

INSERT INTO @ConnectorID 

SELECT DISTINCT SUBSTRING(scpl.Name,17, len(scpl.Name))

    FROM SC_Component as sc 

    INNER JOIN vSMS_SC_Component_PropertyLists scpl on scpl.ID=sc.ID 

    WHERE

    sc.ComponentName LIKE '%Exchange%'

    AND scpl.Name LIKE 'ConnectorConfig%'

 

\-- Get Exchange Server and IsHosted infromation from ConnectorConfig_ property list for each exchange connector 

INSERT INTO @ExchangeConnectorTable 

SELECT DISTINCT conID.ConnectorID, SCPL.Value AS ConfiguredExchangeServer, CAST(SCPL2.Value AS INT) AS IsHosted 

    FROM SC_Component AS SC 

    INNER JOIN vSMS_SC_Component_PropertyLists AS SCPL ON SC.ID=SCPL.ID AND SCPL.ValueIndex=0 

    INNER JOIN vSMS_SC_Component_PropertyLists AS SCPL2 ON SCPL.PropertyListID=SCPL2.PropertyListID AND SCPL2.ValueIndex=1 

    INNER JOIN @ConnectorId conID ON SCPL.Name like '%' + conID.ConnectorID + '%'

    WHERE (SC.ComponentName = 'SMS_EXCHANGE_CONNECTOR' AND SCPL.Name like 'Connector%')

     

\--Get IsPolicyDeployed information from ConfiguredSettings_ property list for each exchange connector 

INSERT INTO @ExchangePolicyTable 

SELECT DISTINCT conID.ConnectorID, (case when SCPL3.Value is NULL THEN 0 ELSE 1 END) AS IsManagedBySCCM 

    FROM SC_Component AS SC 

    INNER JOIN vSMS_SC_Component_PropertyLists AS SCPL3 ON SC.ID=SCPL3.ID 

    INNER JOIN @ConnectorId conID ON SCPL3.Name like '%' + conID.ConnectorID + '%'

    WHERE (SC.ComponentName = 'SMS_EXCHANGE_CONNECTOR' AND SCPL3.Name like 'Configured%')

         

SELECT

(CASE

    WHEN COUNT(*) = 1 THEN NULL

    ELSE COUNT(*)

    END) as DeviceCount,

(CASE

    WHEN eas.ExchangeServer IS NULL THEN NULL

    ELSE (dbo.fnConvertBinaryToBase64String(dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), eas.ExchangeServer ), 'SHA256')))

    END) AS DeviceReportedExchangeServer,

(CASE

    WHEN ect.ConfiguredExchangeServer IS NULL THEN NULL

    ELSE (dbo.fnConvertBinaryToBase64String(dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), ect.ConfiguredExchangeServer ), 'SHA256')))

    END) AS ConfiguredExchangeServer,

ect.IsHosted,

ept.IsPolicyDeployed 

FROM EAS_Property eas 

FULL JOIN @ExchangeConnectorTable ect ON ect.ConfiguredExchangeServer like '%'+ eas.ExchangeServer + '%'

FULL JOIN @ExchangePolicyTable ept ON ept.ConnectorID = ect.ConnectorID 

GROUP BY eas.ExchangeServer, ect.ConfiguredExchangeServer, ect.IsHosted, ept.IsPolicyDeployed 

### Query Including the Unhashed data alongside the hashed data

DECLARE @ConnectorId TABLE

(

    ConnectorID nvarchar(40) not null

)

 

DECLARE @ExchangeConnectorTable TABLE

(

    ConnectorID nvarchar(40) not null,

    ConfiguredExchangeServer nvarchar(256) not null,

    IsHosted int not null

)

 

DECLARE @ExchangePolicyTable TABLE

(

    ConnectorID nvarchar(40) not null,

    IsPolicyDeployed bit not null

 

)

 

\-- Get unique IDs for exchange connectors     

INSERT INTO @ConnectorID 

SELECT DISTINCT SUBSTRING(scpl.Name,17, len(scpl.Name))

    FROM SC_Component as sc 

    INNER JOIN vSMS_SC_Component_PropertyLists scpl on scpl.ID=sc.ID 

    WHERE

    sc.ComponentName LIKE '%Exchange%'

    AND scpl.Name LIKE 'ConnectorConfig%'

 

\-- Get Exchange Server and IsHosted infromation from ConnectorConfig_ property list for each exchange connector 

INSERT INTO @ExchangeConnectorTable 

SELECT DISTINCT conID.ConnectorID, SCPL.Value AS ConfiguredExchangeServer, CAST(SCPL2.Value AS INT) AS IsHosted 

    FROM SC_Component AS SC 

    INNER JOIN vSMS_SC_Component_PropertyLists AS SCPL ON SC.ID=SCPL.ID AND SCPL.ValueIndex=0 

    INNER JOIN vSMS_SC_Component_PropertyLists AS SCPL2 ON SCPL.PropertyListID=SCPL2.PropertyListID AND SCPL2.ValueIndex=1 

    INNER JOIN @ConnectorId conID ON SCPL.Name like '%' + conID.ConnectorID + '%'

    WHERE (SC.ComponentName = 'SMS_EXCHANGE_CONNECTOR' AND SCPL.Name like 'Connector%')

     

\--Get IsPolicyDeployed information from ConfiguredSettings_ property list for each exchange connector 

INSERT INTO @ExchangePolicyTable 

SELECT DISTINCT conID.ConnectorID, (case when SCPL3.Value is NULL THEN 0 ELSE 1 END) AS IsManagedBySCCM 

    FROM SC_Component AS SC 

    INNER JOIN vSMS_SC_Component_PropertyLists AS SCPL3 ON SC.ID=SCPL3.ID 

    INNER JOIN @ConnectorId conID ON SCPL3.Name like '%' + conID.ConnectorID + '%'

    WHERE (SC.ComponentName = 'SMS_EXCHANGE_CONNECTOR' AND SCPL3.Name like 'Configured%')

         

SELECT

(CASE

    WHEN COUNT(*) = 1 THEN NULL

    ELSE COUNT(*)

    END) as DeviceCount,

(CASE

    WHEN eas.ExchangeServer IS NULL THEN NULL

    ELSE (dbo.fnConvertBinaryToBase64String(dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), eas.ExchangeServer ), 'SHA256')))

    END) AS DeviceReportedExchangeServer,

(CASE

    WHEN eas.ExchangeServer IS NULL THEN NULL

    ELSE (eas.ExchangeServer)

    END) AS DeviceReportedExchangeServer,

(CASE

    WHEN ect.ConfiguredExchangeServer IS NULL THEN NULL

    ELSE (dbo.fnConvertBinaryToBase64String(dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), ect.ConfiguredExchangeServer ), 'SHA256')))

    END) AS ConfiguredExchangeServer,

(CASE

    WHEN ect.ConfiguredExchangeServer IS NULL THEN NULL

    ELSE (ect.ConfiguredExchangeServer)

    END) AS ConfiguredExchangeServer,

ect.IsHosted,

ept.IsPolicyDeployed 

FROM EAS_Property eas 

FULL JOIN @ExchangeConnectorTable ect ON ect.ConfiguredExchangeServer like '%'+ eas.ExchangeServer + '%'

FULL JOIN @ExchangePolicyTable ept ON ept.ConnectorID = ect.ConnectorID 

GROUP BY eas.ExchangeServer, ect.ConfiguredExchangeServer, ect.IsHosted, ept.IsPolicyDeployed 

 

 

## TEL_MAM_PolicySettingsStatistics4Deployment2Collection

### Original Query

 

    SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED

    SET NOCOUNT ON

 

    /* Clarification for the return values: 

     MAMPolicyName: the name of MAM policy 

     MAMAppDeploymentCount: How many deployment used this MAM policy 

     DistributeCollectionCount: How many collections are target for this MAM policy 

    */

 

    ;WITH MAM_DCMDIGEST_SIMPLEVERSION AS

    (

        SELECT    ci.CI_ID AS MAMPOLICY_CIID,

            ci.SDMPackageDigest.value(

                    'declare namespace dcm="http://schemas.microsoft.com/SystemsCenterConfigurationManager/2009/07/10/DesiredConfiguration"; 

                     declare namespace name="http://schemas.microsoft.com/SystemsCenterConfigurationManager/2009/06/14/Rules"; 

                     (/dcm:DesiredConfigurationDigest/dcm:AbstractConfigurationItem/name:Annotation/name:DisplayName/@Text)[1]', 'NVARCHAR(MAX)') AS MAMPOLICY_Name             

        FROM    CI_ConfigurationItems ci 

        WHERE    ci.CIType_ID = 70 

    ), ASSIGNMENT_CIS_MAPPING AS

    (

        SELECT    ciAssignment.AssignmentID,

                ciAssignment.ParentAssignmentID,

                ciAssignmentTarget.CI_ID AS FromCI_ID,

                ciRelation.ToCI_ID AS ToCI_ID,

                ciAssignment.AssignmentName,

                ciAssignment.TargetCollectionID 

        FROM CI_CIAssignments ciAssignment 

        INNER JOIN vCI_AssignmentTargetedCIs ciAssignmentTarget ON ciAssignmentTarget.AssignmentID = ciAssignment.AssignmentID 

        INNER JOIN CI_ConfigurationItemRelations ciRelation ON ciAssignmentTarget.CI_ID = ciRelation.FromCI_ID 

        WHERE ciAssignment.ParentAssignmentID IS NOT NULL

    ), MAM_POLICY_MAPPING AS

    (

        SELECT    ciRelation.FromCI_ID AS FromCI_MAMID,

            ciRelation.ToCI_ID AS ToCI_MAMID 

        FROM    CI_ConfigurationItemRelations ciRelation 

        INNER JOIN CI_ConfigurationItems ci 

            ON ciRelation.ToCI_ID = ci.CI_ID 

        WHERE ciRelation.RelationType=23 AND ci.CIType_ID = 70 

    ), MAM_DCMDIGEST AS

    (

        SELECT    mamPolicy.ToCI_MAMID AS MAMPOLICY_CIID,

                mamAssignment.FromCI_ID AS MAMPOLICY_BLID,

                mamAssignment.ToCI_ID AS MAMPOLICY_APPID,

                mamProperty.MAMPOLICY_Name,

                mamAssignment.AssignmentID AS MAMAPP_AssignmentID,

                mamAssignment.ParentAssignmentID AS MAMAPP_ParentAssignmentID,

                mamAssignment.TargetCollectionID AS TargetCollectionID 

        FROM MAM_POLICY_MAPPING mamPolicy 

        INNER JOIN ASSIGNMENT_CIS_MAPPING mamAssignment ON (mamPolicy.FromCI_MAMID = mamAssignment.ToCI_ID)

        INNER JOIN MAM_DCMDIGEST_SIMPLEVERSION mamProperty ON (mamPolicy.ToCI_MAMID = mamProperty.MAMPOLICY_CIID)

    )

    SELECT    dbo.fnConvertBinaryToBase64String(dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), MAMPOLICY_Name), 'SHA256')) AS MAMPolicyName,

            COUNT(MAMAPP_AssignmentID) AS MAMAppDeploymentCount,

            COUNT(TargetCollectionID) AS DistributedCollectionCount 

    FROM MAM_DCMDIGEST 

    GROUP BY MAMPOLICY_Name 

    ORDER BY MAMPOLICY_Name 

 

 

### Query Including the Unhashed data alongside the hashed data

    SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED

    SET NOCOUNT ON

 

    /* Clarification for the return values: 

     MAMPolicyName: the name of MAM policy 

     MAMAppDeploymentCount: How many deployment used this MAM policy 

     DistributeCollectionCount: How many collections are target for this MAM policy 

    */

 

    ;WITH MAM_DCMDIGEST_SIMPLEVERSION AS

    (

        SELECT    ci.CI_ID AS MAMPOLICY_CIID,

            ci.SDMPackageDigest.value(

                    'declare namespace dcm="http://schemas.microsoft.com/SystemsCenterConfigurationManager/2009/07/10/DesiredConfiguration"; 

                     declare namespace name="http://schemas.microsoft.com/SystemsCenterConfigurationManager/2009/06/14/Rules"; 

                     (/dcm:DesiredConfigurationDigest/dcm:AbstractConfigurationItem/name:Annotation/name:DisplayName/@Text)[1]', 'NVARCHAR(MAX)') AS MAMPOLICY_Name             

        FROM    CI_ConfigurationItems ci 

        WHERE    ci.CIType_ID = 70 

    ), ASSIGNMENT_CIS_MAPPING AS

    (

        SELECT    ciAssignment.AssignmentID,

                ciAssignment.ParentAssignmentID,

                ciAssignmentTarget.CI_ID AS FromCI_ID,

                ciRelation.ToCI_ID AS ToCI_ID,

                ciAssignment.AssignmentName,

                ciAssignment.TargetCollectionID 

        FROM CI_CIAssignments ciAssignment 

        INNER JOIN vCI_AssignmentTargetedCIs ciAssignmentTarget ON ciAssignmentTarget.AssignmentID = ciAssignment.AssignmentID 

        INNER JOIN CI_ConfigurationItemRelations ciRelation ON ciAssignmentTarget.CI_ID = ciRelation.FromCI_ID 

        WHERE ciAssignment.ParentAssignmentID IS NOT NULL

    ), MAM_POLICY_MAPPING AS

    (

        SELECT    ciRelation.FromCI_ID AS FromCI_MAMID,

            ciRelation.ToCI_ID AS ToCI_MAMID 

        FROM    CI_ConfigurationItemRelations ciRelation 

        INNER JOIN CI_ConfigurationItems ci 

            ON ciRelation.ToCI_ID = ci.CI_ID 

        WHERE ciRelation.RelationType=23 AND ci.CIType_ID = 70 

    ), MAM_DCMDIGEST AS

    (

        SELECT    mamPolicy.ToCI_MAMID AS MAMPOLICY_CIID,

                mamAssignment.FromCI_ID AS MAMPOLICY_BLID,

                mamAssignment.ToCI_ID AS MAMPOLICY_APPID,

                mamProperty.MAMPOLICY_Name,

                mamAssignment.AssignmentID AS MAMAPP_AssignmentID,

                mamAssignment.ParentAssignmentID AS MAMAPP_ParentAssignmentID,

                mamAssignment.TargetCollectionID AS TargetCollectionID 

        FROM MAM_POLICY_MAPPING mamPolicy 

        INNER JOIN ASSIGNMENT_CIS_MAPPING mamAssignment ON (mamPolicy.FromCI_MAMID = mamAssignment.ToCI_ID)

        INNER JOIN MAM_DCMDIGEST_SIMPLEVERSION mamProperty ON (mamPolicy.ToCI_MAMID = mamProperty.MAMPOLICY_CIID)

    )

    SELECT    dbo.fnConvertBinaryToBase64String(dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), MAMPOLICY_Name), 'SHA256')) AS MAMPolicyName, MAMPOLICY_Name,

            COUNT(MAMAPP_AssignmentID) AS MAMAppDeploymentCount,

            COUNT(TargetCollectionID) AS DistributedCollectionCount 

    FROM MAM_DCMDIGEST 

    GROUP BY MAMPOLICY_Name 

    ORDER BY MAMPOLICY_Name 

 

## TEL_Perf_TableSize

### Original Query

SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED

 

SELECT TOP 100 

dbo.fnConvertBinaryToBase64String(

dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), a2.name), 'SHA256')

) AS Table_name,

a1.rows as Records,

(a1.reserved)* 8 AS Reserved_kb,

a1.data * 8 AS Data_kb,

(CASE WHEN (a1.used) > a1.data THEN (a1.used) - a1.data ELSE 0 END) * 8 AS Indexes_kb 

FROM (SELECT ps.object_id,

SUM (CASE WHEN (ps.index_id < 2) THEN row_count ELSE 0 END) AS [rows],

SUM (ps.reserved_page_count) AS reserved,

SUM (CASE WHEN (ps.index_id < 2)

THEN (ps.in_row_data_page_count + ps.lob_used_page_count + ps.row_overflow_used_page_count)

ELSE (ps.lob_used_page_count + ps.row_overflow_used_page_count) END) AS data,

SUM (ps.used_page_count) AS Used 

FROM sys.dm_db_partition_stats ps 

GROUP BY ps.object_id

) AS a1 

INNER JOIN sys.all_objects a2 ON (a1.object_id = a2.object_id)

INNER JOIN sys.schemas a3 ON (a2.schema_id = a3.schema_id)

WHERE a2.type <> N'S' and a2.type <> N'IT'

ORDER BY a1.data DESC

 

### Query Including the Unhashed data alongside the hashed data

SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED

 

SELECT TOP 100 

dbo.fnConvertBinaryToBase64String(

dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), a2.name), 'SHA256')

) AS Table_name, a2.name as Unhashed_Table_Name,

a1.rows as Records,

(a1.reserved)* 8 AS Reserved_kb,

a1.data * 8 AS Data_kb,

(CASE WHEN (a1.used) > a1.data THEN (a1.used) - a1.data ELSE 0 END) * 8 AS Indexes_kb 

FROM (SELECT ps.object_id,

SUM (CASE WHEN (ps.index_id < 2) THEN row_count ELSE 0 END) AS [rows],

SUM (ps.reserved_page_count) AS reserved,

SUM (CASE WHEN (ps.index_id < 2)

THEN (ps.in_row_data_page_count + ps.lob_used_page_count + ps.row_overflow_used_page_count)

ELSE (ps.lob_used_page_count + ps.row_overflow_used_page_count) END) AS data,

SUM (ps.used_page_count) AS Used 

FROM sys.dm_db_partition_stats ps 

GROUP BY ps.object_id

) AS a1 

INNER JOIN sys.all_objects a2 ON (a1.object_id = a2.object_id)

INNER JOIN sys.schemas a3 ON (a2.schema_id = a3.schema_id)

WHERE a2.type <> N'S' and a2.type <> N'IT'

ORDER BY a1.data DESC

 

## TEL_SetupInfo

### Original Query

    \-- nolock for scope of procedure 

    SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED

    SET NOCOUNT ON

    DECLARE @LicenseType INT

    DECLARE @Version Nvarchar(20)

    DECLARE @SysCenterId Nvarchar(MAX)

    DECLARE @TenantId Nvarchar(MAX)

    DECLARE @TelemetryLevel INT

    DECLARE @OfflineMode INT

    DECLARE @SiteNumber INT = dbo.fnGetSiteNumber();

 

    select top 1 @LicenseType=value1 from SetupInfo where id=N'Type'

select top 1 @Version=string1 from SetupInfo where id=N'VERSION'

    select top 1 @SysCenterId=string1 from SetupInfo where id=N'SYSCENTERID'

 

    select @TelemetryLevel =

        scp.Value3 

     FROM SC_Component sc 

     INNER JOIN SC_Component_Property scp ON scp.ComponentID = sc.ID 

     WHERE

        sc.SiteNumber = @SiteNumber 

        AND sc.ComponentName = 'SMS_REPLICATION_CONFIGURATION_MONITOR'

        AND scp.Name = 'TelemetryLevel';

 

     select @TenantId = dbo.fnConvertBinaryToBase64String(dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), [dbo].[fnGetHierarchyID]()), 'SHA256') )

     

     select @OfflineMode = scp.Value3 from SC_SysResUse sc inner join SC_SysResUse_Property scp on sc.ID = scp.SysResUseID Where sc.RoleTypeID = 23 and (Name = 'OfflineMode')

     select @LicenseType as LicenseType, @Version as Version, @SysCenterId as SysCenterId, @TelemetryLevel as TelemetryLevel, @TenantId as TenantId, isnull(@OfflineMode,0) as OfflineMode 

 

### Query Including the Unhashed data alongside the hashed data

    \-- nolock for scope of procedure 

    SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED

    SET NOCOUNT ON

    DECLARE @LicenseType INT

    DECLARE @Version Nvarchar(20)

    DECLARE @SysCenterId Nvarchar(MAX)

    DECLARE @TenantId Nvarchar(MAX)

    DECLARE @TelemetryLevel INT

    DECLARE @OfflineMode INT

    DECLARE @SiteNumber INT = dbo.fnGetSiteNumber();

 

    select top 1 @LicenseType=value1 from SetupInfo where id=N'Type'

select top 1 @Version=string1 from SetupInfo where id=N'VERSION'

    select top 1 @SysCenterId=string1 from SetupInfo where id=N'SYSCENTERID'

 

    select @TelemetryLevel =

        scp.Value3 

     FROM SC_Component sc 

     INNER JOIN SC_Component_Property scp ON scp.ComponentID = sc.ID 

     WHERE

        sc.SiteNumber = @SiteNumber 

        AND sc.ComponentName = 'SMS_REPLICATION_CONFIGURATION_MONITOR'

        AND scp.Name = 'TelemetryLevel';

 

     select @TenantId = dbo.fnConvertBinaryToBase64String(dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), [dbo].[fnGetHierarchyID]()), 'SHA256') )

     

     select @OfflineMode = scp.Value3 from SC_SysResUse sc inner join SC_SysResUse_Property scp on sc.ID = scp.SysResUseID Where sc.RoleTypeID = 23 and (Name = 'OfflineMode')

     select @LicenseType as LicenseType, @Version as Version, @SysCenterId as SysCenterId, @TelemetryLevel as TelemetryLevel, @TenantId as TenantId, [dbo].[fnGetHierarchyID]() as Unhashed_HierarchyID, isnull(@OfflineMode,0) as OfflineMode 

## TEL_SQL_DBSchema

### Original Query

 

SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED

SET NOCOUNT ON

 

/* Clarification for the return values: 

Schema version and whether customer did any customization 

*/

 

\-- Setup Version 

SELECT dbo.fnConvertBinaryToBase64String(

dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), DS.ObjectName), 'SHA256')

) AS ObjectNameHash,

DS.ObjectVersion AS ObjectVersion,

DS.UpdatedBy AS UpdatedBy,

DS.ObjectHash As ObjectHash 

FROM dbo.DBSchema DS 

INNER JOIN SC_SiteDefinition SS 

ON DS.SiteNumber = SS.SiteNumber 

WHERE ISNULL(SS.parentsitecode, N'') = N''

 

 

### Query Including the Unhashed data alongside the hashed data

SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED

SET NOCOUNT ON

 

/* Clarification for the return values: 

Schema version and whether customer did any customization 

*/

 

\-- Setup Version 

SELECT dbo.fnConvertBinaryToBase64String(

dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), DS.ObjectName), 'SHA256')

) AS ObjectNameHash, DS.ObjectName,

DS.ObjectVersion AS ObjectVersion,

DS.UpdatedBy AS UpdatedBy,

DS.ObjectHash As ObjectHash 

FROM dbo.DBSchema DS 

INNER JOIN SC_SiteDefinition SS 

ON DS.SiteNumber = SS.SiteNumber 

WHERE ISNULL(SS.parentsitecode, N'') = N''

 

## Identifier at the end of each Results entry

### Original Query

Not Applicable

### Query Including the unhashed data alongside the hashed data

Declare @tenantid as nvarchar(max)

     select @TenantId = dbo.fnConvertBinaryToBase64String(dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), [dbo].[fnGetHierarchyID]()), 'SHA256') )

     Declare @hierarchyid as nvarchar(max)

     select @hierarchyid = [dbo].[fnGetHierarchyID]()

     select @hierarchyid, @tenantid

Enjoy.  
"The M in WMI stands for Magic"  
"Everyone is an expert at something" Kim Oppalfens – Enterprise client management Expert for lack of any other expertise  
Enterprise Client Management MVP  






 

[1]: /Blog/Post/5/Configuration-Manager-telemetry---usage-metrics-(work-in-progress)

  
 Night Mode