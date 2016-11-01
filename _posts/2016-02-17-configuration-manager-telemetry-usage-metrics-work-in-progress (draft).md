---
title: "Configuration Manager telemetry, usage metrics, work in progress"
author: Tom Degreef
categories:
  - SCCM
tags:
  - SCCM
---

Telemetry, what is it about?

Microsoft has quite a bit of information here about its new telemetry data system for SCCM here: [https://technet.microsoft.com/en-US/library/mt613113.aspx ](https://technet.microsoft.com/en-US/library/mt613113.aspx )

Below are my findings and additions to that documentation that people are inquiring about, but let me start off with the why, of it all. The new Configuration Manager comes with a brand new servicing mechanism. You should be aware by now that Windows 10 comes with a pretty high release cadence (a new Windows every 4 months). To keep up with that pace, Configuration Manager is planned to follow suit, and more or less follow that same cadence. Now, quite some people are sceptic about that increased cadence and the impact on the different products quality. To answer the challenges that come with this increased pace Microsoft plans to ship fast / fix fast, and that's were telemetry comes in.

The general idea is to find the setups and "modus operandi" that are frequently used by a large set of customers and use those in testing. Additionally, the telemetry data should prove to be a shortcut to get troubleshooting data to the product team. In other words, sharing data on the way you use the product should be in your own interest.

Sounds good, but am I supposed to just trust Microsoft in collecting data from my environment that might be privacy sensitive? Well, yes and no. Microsoft takes privacy extremely serious, and so does the Configuration Manager product team. The product team has, imho, very good reasons to make sure the data they collect isn't catalogued as privacy sensitive, as doing so would introduce them to a drastically increased involvement of both legal and auditing, as for any Microsoft service that holds privacy sensitive data. To keep the level of scrutiny they'd have to go through in check, the ConfigMgr team starts off with anonymizing the data. They do that by hashing some of the data, so that any privacy sensitive data isn't readable to them.

And that's where some of the challenges come in for customers that are privacy sensitive and/or have auditing breath down their neck. This hashed data isn't readable to them either, which worries some of them, as they don't know what they are sending out. Below I'll explain, for all the hashed values I found in telemetry results so far how you can make the data human readable alongside the hash so that you can control what the data you are sending out actually means.

## Tables  

The **telemetry table** contains the names and id's of the stored procedures that are responsible for collecting the telemetry data. In the environments that I've verified this on, there are 150 stored procedures lists in the telemetry table. You can have a look at this info by running the following query

```sql
SELECT *
FROM Telemetry
ORDER BY NAME
```

The results are stored in a table called TEL_telemetryresults. You can look at your own results by running the following query

```sql
SELECT *
FROM TEL_TelemetryResults
```

Depending upon the level of data you've chosen you should see a number of rows returned. There should be one thing that catches your eye quite swiftly. As you can see in the screenshot below each row has a results column that ends with a returning hash. Which opens up the very first question, what is this hash all about?  

Well this particular hash is used to correlate data between the different rows in your telemetry results so the product team can store all data coming from one customer together. Given the introduction they need a way to do that without making your company name or anything similar that could identify your environment, and hence they need to anonymize the data. Now, every Configuration Manager environment has a randomly generated hierarchyid that could be used for this purpose. But even that wasn't anoynymous enough for the Configuration Manager product team. To anonymize the data they've chosen to hash that hierarchy id using *SHA256*.

![][1]

You can get your own hierarchy id and the accompanying hash to validate this data by running the following query:  

```sql
DECLARE @CMid AS NVARCHAR(max)

SELECT @CMId = dbo.fnConvertBinaryToBase64String(
dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), [dbo].fnGetHierarchyID()), 'SHA256'))

DECLARE @hierarchyid AS NVARCHAR(max)

SELECT @hierarchyid = [dbo].[fnGetHierarchyID]()

SELECT @hierarchyid AS Hierarchyid
	,@CMId AS Hash
```


### Stored procedures  

There are a bunch of stored procedures involved in collecting the telemetry data, and most of them just generate just 1 of the rows in the telemetryresults table. You can find the stored procedures responsible for collecting data by running the following query.

```sql
SELECT DISTINCT o.NAME AS 'Stored Procedures'
	,o.*
FROM SYSOBJECTS o
INNER JOIN SYSCOMMENTS c ON o.id = c.id
WHERE o.NAME LIKE 'tel_%'
	AND o.xtype = 'P'
```

If you're only interested in the ones that generate data for the **telemetryresults** table run the query below.

```sql
SELECT DISTINCT o.NAME AS 'Stored Procedures',o.*
FROM SYSOBJECTS o
INNER JOIN SYSCOMMENTS c ON o.id = c.id
WHERE o.name
like 'TEL_%' and o.xtype = 'P' and o.name in (select name from Telemetry)
```

You could subsequently analyze the stored procedures to see what it is they are collecting, but that is an elaborate exercise. As we've seen that SHA256 is the hashing mechanism of choice I've chosen to check which of these stored procedures use the SHA256 function. I've identified the stored procedures, and linked id's using this query

```sql
SELECT DISTINCT o.NAME AS Object_Name
	,Telemetry.id
	,o.type_desc
	,m.DEFINITION
FROM sys.sql_modules m
INNER JOIN sys.objects o ON m.object_id = o.object_id
INNER JOIN telemetry ON o.NAME = Telemetry.NAME
WHERE m.DEFINITION LIKE '%sha256%'
	AND o.NAME LIKE 'tel_%'
```

This results in the following list of id's

| Object Name                    | Id                                   |
| ------------------------------ | ------------------------------------ |
| TEL_SQL_DBSchema               | 0F40B971-AAC7-4A39-8CDA-1E023C833306 |
| TEL_App_Requirements           | A0764DB2-8F8B-495A-A143-1F20A58E7A5C |
| TEL_SetupInfo                  | E1201168-0A70-41B7-857E-309F8A5FB96B |
| TEL_Content_Package            | 69FC4B89-3561-4360-9157-4F8E896F7FB9 |
| TEL_Perf_TableSize             | 3B694B4A-DA65-4E60-BAE9-5796849A9586 |
| TEL_ConfigPack_AssignmentCount | 79861B77-8313-4009-ABF3-5819ABFE4666 |
| TEL_Content_DPState            | ACABF386-BCD1-48C5-9C7F-A33DADA6E89D |
| TEL_DCM_BuiltinSettings        | FDC2F647-FE63-471D-903F-AC6DE54F5F58 |
| TEL_EAS_Connectors             | 942B1F7E-EB3F-4576-8CB8-F8066D31940F |

Which in turn lets you focus on the **telemeteryresults **table and the rows that contain hashed information:

```sql
SELECT *
FROM TEL_TelemetryResults
WHERE id IN (
		'ACABF386-BCD1-48C5-9C7F-A33DADA6E89D'
		,'69FC4B89-3561-4360-9157-4F8E896F7FB9'
		,'2E8CC4FA-738D-4A48-B36F-E981344C97C3'
		,'942B1F7E-EB3F-4576-8CB8-F8066D31940F'
		,'CD6B1D69-5F70-46B1-BC82-2C99764188B5'
		,'3B694B4A-DA65-4E60-BAE9-5796849A9586'
		,'E1201168-0A70-41B7-857E-309F8A5FB96B'
		,'0F40B971-AAC7-4A39-8CDA-1E023C833306'
		)
```

Or on those that should not contain any hashed information by changing the where clause to use not in instead of in. This should allow you to quickly check whether the results column still has data you can't understand. (Should that be the case feel free to share the ID of the row and I'll happily look into it.)


The last ID '0F40B971-AAC7-4A39-8CDA-1E023C833306' contains the full schema of your Configuration Manager database as collected by the **TEL_SQL_DBSCHEMA** stored procedure. When you look at the stored procedure definition you'll notice that it runs the following query to collect the data:

```sql  
SELECT dbo.fnConvertBinaryToBase64String(dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), DS.ObjectName), 'SHA256')) AS ObjectNameHash
	,DS.ObjectVersion AS ObjectVersion
	,DS.UpdatedBy AS UpdatedBy
	,DS.ObjectHash AS ObjectHash
FROM dbo.DBSchema DS
INNER JOIN SC_SiteDefinition SDef ON DS.SiteNumber = SDef.SiteNumber
WHERE ISNULL(SDef.parentsitecode, N'') = N''  
```

As should be apparent, the objectnames are obfuscated in this stored procedure. Should you like to know what the obfuscated data really means you can modify the query slightly and add another item in the select section of the query to include the data before it is hashed like so:


```sql  
SELECT DS.ObjectName
	,dbo.fnConvertBinaryToBase64String(dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), DS.ObjectName), 'SHA256')) AS ObjectNameHash
	,DS.ObjectVersion AS ObjectVersion
	,DS.UpdatedBy AS UpdatedBy
	,DS.ObjectHash AS ObjectHash
FROM dbo.DBSchema DS
INNER JOIN SC_SiteDefinition SS ON DS.SiteNumber = SS.SiteNumber
WHERE ISNULL(SS.parentsitecode, N'') = N'' 
```  


As you can see, all I did was include the column DS.ObjectName before it was hashed so you could see it in readable format alongside the hashed format. The reason they hash the data in this particular instance is because your're schema could contain your company name, or other privacy sensitive data. The most likely way this would end up in your schema is by including that information in the names of your custom hardware inventory classes.

This is just one of the 8 queries that might contain hashed data, but the mechanism above is repeatable for the other stored procedures. I'll add the queries needed to represent the cleartext data and the hashed variant over the next couple of days.

Enjoy.  
"The M in WMI stands for Magic"  
"Everyone is an expert at something" Kim Oppalfens - Enterprise Mobility Expert for lack of any other expertise  

[1]: http://i.imgur.com/PakPsmw.png

