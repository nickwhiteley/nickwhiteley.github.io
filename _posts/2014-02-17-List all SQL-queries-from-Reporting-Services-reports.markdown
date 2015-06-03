---
layout: post
title:  "List all SQL queries from Reporting Services reports"
date:   2014-02-17 09:21:01
categories: Reporting Services
---

Here is a quick way to pull out the SQL for all reports in an instance of Microsoft's Reporting Services.  I needed a quick way to get the queries (commandtext) for 350 or so reports and I didn't want to use Visual Studio to open them all individually.

I added all of the field names too as this can make it quicker to scan through for relevant tables for experienced analysts.  The results can then be pasted into Excel.

I found some SqlText column entries were getting truncated (suspiciously at 256 characters) but not all.  We all have our own approach to these sorts of problems but mine was to save the result set into a permanent table and use Excel's data connections to pull the data into a tab.

The query is slow (5 minutes for 350 reports).

Tested against Reporting Services 2008.

{% highlight sql %}
;WITH XMLNAMESPACES (
 DEFAULT 
 'http://schemas.microsoft.com/sqlserver/reporting/2008/01/reportdefinition',
 'http://schemas.microsoft.com/SQLServer/reporting/reportdesigner' AS rd
)
 SELECT
  path,
  name,
  d.value('@Name[1]', 'VARCHAR(50)') AS DataSetName,
  fn.value('@Name[1]', 'VARCHAR(50)') AS ReportFieldName,
  ct.value('CommandText[1]', 'VARCHAR(MAX)') AS SqlText
 INTO #reports
 FROM (
  SELECT REPLACE(Path, '~~', '') as Path,
  name, 
  CAST(CAST(content AS VARBINARY(MAX)) AS XML) AS reportXML
  FROM ReportServer.dbo.Catalog
  WHERE Type = 2
 ) a
 CROSS APPLY reportXML.nodes('/Report/DataSets/DataSet') r(d)
 CROSS APPLY d.nodes('Query') q(ct)
 CROSS APPLY d.nodes('Fields/Field') f(fn)


SELECT DISTINCT Path, Name, DataSetName,  (
     SELECT ReportFieldName + ', '
     FROM #reports r
     WHERE r.Path = rr.path
   AND r.DataSetName = rr.DataSetName
     FOR XML PATH ('')
     ) as Fields,
       SqlText
FROM #reports rr

{% endhighlight %}

