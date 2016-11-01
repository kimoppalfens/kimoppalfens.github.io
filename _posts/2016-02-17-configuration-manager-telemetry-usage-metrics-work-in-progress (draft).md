Telemetry, what is it about?
<h1>Telemetry explained</h1>
<p>Microsoft has quite a bit of information here about its new telemetry data system for SCCM here: <a href="https://technet.microsoft.com/en-US/library/mt613113.aspx">https://technet.microsoft.com/en-US/library/mt613113.aspx</a>
	</p>
<p>Below are my findings and additions to that documentation that people are inquiring about, but let me start off with the why, of it all. The new Configuration Manager comes with a brand new servicing mechanism. You should be aware by now that Windows 10 comes with a pretty high release cadence (a new Windows every 4 months). To keep up with that pace, Configuration Manager is planned to follow suit, and more or less follow that same cadence. Now, quite some people are sceptic about that increased cadence and the impact on the different products quality. To answer the challenges that come with this increased pace Microsoft plans to ship fast / fix fast, and that's were telemetry comes in.</p>
<p>The general idea is to find the setups and "modus operandi" that are frequently used by a large set of customers and use those in testing. Additionally, the telemetry data should prove to be a shortcut to get troubleshooting data to the product team. In other words, sharing data on the way you use the product should be in your own interest.</p>
<p>Sounds good, but am I supposed to just trust Microsoft in collecting data from my environment that might be privacy sensitive? Well, yes and no. Microsoft takes privacy extremely serious, and so does the Configuration Manager product team. The product team has, imho, very good reasons to make sure the data they collect isn't catalogued as privacy sensitive, as doing so would introduce them to a drastically increased involvement of both legal and auditing, as for any Microsoft service that holds privacy sensitive data. To keep the level of scrutiny they'd have to go through in check, the ConfigMgr team starts off with anonymizing the data. They do that by hashing some of the data, so that any privacy sensitive data isn't readable to them.</p>
<p>And that's where some of the challenges come in for customers that are privacy sensitive and/or have auditing breath down their neck. This hashed data isn't readable to them either, which worries some of them, as they don't know what they are sending out. Below I'll explain, for all the hashed values I found in telemetry results so far how you can make the data human readable alongside the hash so that you can control what the data you are sending out actually means.</p>
<h1>Database objects involved<br />
</h1>
<h2>Tables<br />
</h2>
<p>The <strong>telemetry table</strong> contains the names and id's of the stored procedures that are responsible for collecting the telemetry data. In the environments that I've verified this on, there are 150 stored procedures lists in  the telemetry table. You can have a look at this info by running the following query</p>

```sql
select * from Telemetry order by name
```


<style="margin-left: 36pt">
<span style="font-family:Consolas; font-size:9pt"><br />
		</span>
<p>The results are stored in a table called  TEL_telemetryresults. You can look at your own results by running the following query</p>

```sql
select * from TEL_TelemetryResults
```

<span style="font-style:normal;" depending on your environment.</span><br />
<p>Depending upon the level of data you've chosen you should see a number of rows returned. There should be one thing that catches your eye quite swiftly. As you can see in the screenshot below each row has a results column that ends with a returning hash. Which opens up the very first question, what is this hash all about?<br />
</p>
<p><span style="font-style:normal;">Well this particular hash is used to correlate data between the different rows in your telemetry results so the product team can store all data coming from one customer together. Given the introduction they need a way to do that without making your company name or anything similar that could identify your environment, and hence they need to anonymize the data. Now, every Configuration Manager environment has a randomly generated hierarchyid that could be used for this purpose. But even that wasn't anoynymous enough for the Configuration Manager product team. To anonymize the data they've chosen to hash that hierarchy id using <strong>SHA256.<br />
</strong></span></p>
<a href="http://i.imgur.com/PakPsmw.png"><img src="http://i.imgur.com/PakPsmw.png"></a>


<p><span>You can get your own hierarchy id and the accompanying hash to validate this data by running the following query:<br />
</span></p>
```sql
Declare @tenantid as
nvarchar(max)

select @TenantId = dbo.fnConvertBinaryToBase64String(dbo.fnMDMCalculateHash(CONVERT(VARBINARY(MAX), [dbo].fnGetHierarchyID),
'SHA256')
)

Declare @hierarchyid as nvarchar(max)
select @hierarchyid = [dbo].[fnGetHierarchyID]()
select @hierarchyid, @tenantid
```

</span></span></p>
<p>
 </p>
<h3>Stored procedures<br />
</h3>
<p>There are a bunch of stored procedures involved in collecting the telemetry data, and most of them just generate just 1 of the rows in the telemetryresults table. You can find the stored procedures responsible for collecting data by running the following query.</p>
```SQL
SELECT distinct o.name As 'Stored Procedures',o.*
FROM SYSOBJECTS o 
INNER JOIN SYSCOMMENTS c
ON o.id = c.id WHERE o.name like 'tel_%' and o.xtype = 'P'
```
<p>
 </p>
<p>If you're only interested in the ones that generate data for the <strong>telemetryresults</strong> table run the query below.</p>
```SQL
SELECT distinct o.name As Stored Procedures',o.*
FROM SYSOBJECTS o 
INNER JOIN SYSCOMMENTS c
ON o.id = c.id
WHERE o.name like 'tel_%' and o.xtype = 'P' and o.name in (select name from Telemetry)
```
<p>You could subsequently analyze the stored procedures to see what it is they are collecting, but that is an elaborate exercise. As we've seen that SHA256 is the hashing mechanism of choice I've chosen to check which of these stored procedures use the SHA256 function. I've identified the stored procedures, and linked id's using this query</p>
```SQL
SELECT DISTINCT o.name AS Object_Name, Telemetry.id, o.type_desc, m.definition 
FROM sys.sql_modules m 
INNER JOIN sys.objects o ON m.object_id = o.object_id
INNER join telemetry on o.name = Telemetry.Name
where m.definition like '%sha256%' and o.name like 'tel_%'
``` 


<p>
 </p>
<p>This results in the following list of id's</p></p>


|Object Name | Id |
|:--------|:-------:|
|TEL_SQL_DBSchema	|0F40B971-AAC7-4A39-8CDA-1E023C833306
|TEL_App_Requirements	|A0764DB2-8F8B-495A-A143-1F20A58E7A5C
|TEL_SetupInfo	|E1201168-0A70-41B7-857E-309F8A5FB96B
|TEL_Content_Package|	69FC4B89-3561-4360-9157-4F8E896F7FB9
|TEL_Perf_TableSize|	3B694B4A-DA65-4E60-BAE9-5796849A9586
|TEL_ConfigPack_AssignmentCount|	79861B77-8313-4009-ABF3-5819ABFE4666
|TEL_Content_DPState|	ACABF386-BCD1-48C5-9C7F-A33DADA6E89D
|TEL_DCM_BuiltinSettings|	FDC2F647-FE63-471D-903F-AC6DE54F5F58
|TEL_EAS_Connectors|	942B1F7E-EB3F-4576-8CB8-F8066D31940F

<p>Which in turn lets you focus on the <strong>telemeteryresults </strong>table and the rows that contain hashed information:</p>
```SQL
select
*
from TEL_TelemetryResults
where id in
('ACABF386-BCD1-48C5-9C7F-A33DADA6E89D',
--TEL_Content_DPState
'69FC4B89-3561-4360-9157-4F8E896F7FB9',
--TEL_Content_Package
'2E8CC4FA-738D-4A48-B36F-E981344C97C3',
--TEL_DCM_BuiltinSettings
'942B1F7E-EB3F-4576-8CB8-F8066D31940F',
--TEL_EAS_Connectors
'CD6B1D69-5F70-46B1-BC82-2C99764188B5',
--TEL_MAM_PolicySettingStatistics4Deployment2Collection
'3B694B4A-DA65-4E60-BAE9-5796849A9586',
--TEL_Perf_TableSize
'E1201168-0A70-41B7-857E-309F8A5FB96B',
--TEL_SetupInfo
'0F40B971-AAC7-4A39-8CDA-1E023C833306'
)
--TEL_SQL_DBSchema
```
<p>Or on those that should not contain any hashed information by changing the where clause to use not in instead of in.  This should allow you to quickly check whether the results column still has data you can't understand. (Should that be the case feel free to share the ID of the row and I'll happily look into it.)</p>
<p>
 </p>
<h1>Obfuscated data / data hashing and making it human readable again<br />
</h1>
<p>The last ID <span style="color:red; font-family:Consolas; font-size:9pt">'0F40B971-AAC7-4A39-8CDA-1E023C833306' </span>contains the full schema of your Configuration Manager database as collected by the <strong>TEL_SQL_DBSCHEMA</strong> stored procedure. When you look at the stored procedure definition you'll notice that it runs the following query to collect the data:</p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue; background-color:yellow">SELECT</span><span style="background-color:yellow"> dbo<span style="color:gray">.</span>fnConvertBinaryToBase64String<span style="color:gray">(</span></span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="background-color:yellow">dbo<span style="color:gray">.</span>fnMDMCalculateHash<span style="color:gray">(<span style="color:fuchsia">CONVERT<span style="color:gray">(<span style="color:blue">VARBINARY<span style="color:gray">(<span style="color:fuchsia">MAX<span style="color:gray">),</span> DS<span style="color:gray">.</span>ObjectName<span style="color:gray">),</span><br />
										<span style="color:red">'SHA256'<span style="color:gray">))</span><br />
											<span style="color:blue">AS</span> ObjectNameHash<span style="color:gray">,</span></span></span></span></span></span></span></span></span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt">           DS<span style="color:gray">.</span>ObjectVersion <span style="color:blue">AS</span> ObjectVersion<span style="color:gray">,</span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt">           DS<span style="color:gray">.</span>UpdatedBy <span style="color:blue">AS</span> UpdatedBy<span style="color:gray">,</span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt">           DS<span style="color:gray">.</span>ObjectHash <span style="color:blue">As</span> ObjectHash<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue">FROM</span>   dbo<span style="color:gray">.</span>DBSchema DS<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:gray">INNER</span><br />
			<span style="color:gray">JOIN</span> SC_SiteDefinition SS<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue">ON</span> DS<span style="color:gray">.</span>SiteNumber <span style="color:gray">=</span> SS<span style="color:gray">.</span>SiteNumber<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue">WHERE</span><br />
			<span style="color:fuchsia">ISNULL<span style="color:gray">(</span>SS<span style="color:gray">.</span>parentsitecode<span style="color:gray">,</span><br />
				<span style="color:red">N''<span style="color:gray">)</span><br />
					<span style="color:gray">=</span> N''<br />
</span></span></span></p>
<p>
 </p>
<p>As should be apparent, the objectnames are obfuscated in this stored procedure. Should you like to know what the obfuscated data really means you can modify the query slightly and another item in the select section of the query to include the data before it is hashed like so:</p>
<p>
 </p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue; background-color:yellow">SELECT</span><span style="background-color:yellow"> DS<span style="color:gray">.</span>ObjectName<span style="color:gray">,</span> dbo<span style="color:gray">.</span>fnConvertBinaryToBase64String<span style="color:gray">(</span></span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="background-color:yellow">dbo<span style="color:gray">.</span>fnMDMCalculateHash<span style="color:gray">(<span style="color:fuchsia">CONVERT<span style="color:gray">(<span style="color:blue">VARBINARY<span style="color:gray">(<span style="color:fuchsia">MAX<span style="color:gray">),</span> DS<span style="color:gray">.</span>ObjectName<span style="color:gray">),</span><br />
										<span style="color:red">'SHA256'<span style="color:gray">))</span><br />
											<span style="color:blue">AS</span> ObjectNameHash<span style="color:gray">,</span></span></span></span></span></span></span></span></span><strong><br />
			</strong></span></p>
<p><span style="font-family:Consolas; font-size:9pt">      DS<span style="color:gray">.</span>ObjectVersion <span style="color:blue">AS</span> ObjectVersion<span style="color:gray">,</span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt">        DS<span style="color:gray">.</span>UpdatedBy <span style="color:blue">AS</span> UpdatedBy<span style="color:gray">,</span><br />
		</span></p>
<p><span style="font-family:Consolas; font-size:9pt">           DS<span style="color:gray">.</span>ObjectHash <span style="color:blue">As</span> ObjectHash<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue">FROM</span>   dbo<span style="color:gray">.</span>DBSchema DS<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:gray">INNER</span><br />
			<span style="color:gray">JOIN</span> SC_SiteDefinition SS<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue">ON</span> DS<span style="color:gray">.</span>SiteNumber <span style="color:gray">=</span> SS<span style="color:gray">.</span>SiteNumber<br />
</span></p>
<p><span style="font-family:Consolas; font-size:9pt"><br />
			<span style="color:blue">WHERE</span><br />
			<span style="color:fuchsia">ISNULL<span style="color:gray">(</span>SS<span style="color:gray">.</span>parentsitecode<span style="color:gray">,</span><br />
				<span style="color:red">N''<span style="color:gray">)</span><br />
					<span style="color:gray">=</span> N''</span><br />
			</span><br />
		</span></p>
<p>
 </p>
<p>As you can see, all I did was include the column DS.ObjectName before it was hashed so you could see it in readable format alongside the hashed format. The reason they hash the data in this particular instance is because your're schema could contain your company name, or other privacy sensitive data. The most likely way this would end up in your schema is by including that information in the names of your custom hardware inventory classes.</p>
<p>This is just one of the 8 queries that might contain hashed data, but the mechanism above is repeatable for the other stored procedures. I'll add the queries needed to represent the cleartext data and the hashed variant over the next couple of days.</p>
<p><span style="color:#555555; font-family:Arial; font-size:9pt">Enjoy.<br />"The M in WMI stands for Magic"<br />""Everyone is an expert at something" Kim Oppalfens - ConfigMgr Expert for lack of any other expertise<br />System Center Configuration Manager MVP<br />http://www.scug.be/thewmiguy/default.aspx</p>
<p>http://www.linkedin.com/in/kimoppalfens</p>
<p>http://twitter.com/thewmiguy</span></p>
