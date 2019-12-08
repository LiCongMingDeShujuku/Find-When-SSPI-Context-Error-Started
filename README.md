![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 查找SSPI Context Error开始的时间
#### Find When SSPI Context Error Started
**发布-日期: 2017年11月22日 (评论)**

![#](images/find-when-sspi-context-error-started-1.png?raw=true "#")

## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
正如我们大多数人所知道的，获取SSPI Context error 是非常烦人的，并且令人愤怒的是，这仍然会发生在最新的服务器上。
在大多数情况下，这是由（服务主体名称）问题引起的。不幸的是，虽然这是在与数据库服务器的“连接”时发现的，但它的自身问题只能通过非数据库中心操作来解决，例如通过管理网络安全层上的服务主体名称/ DNS条目。
一旦纠正，将需要重新启动数据库服务，使其正确分配新的SPN。
我以下的逻辑（logic）将会向你展示，如何开始查找问题发生的时间。如上图所示，它会给你展示所有的日志（logs），方便你查看。我写了一个快速的WHERE语句，可以帮助你开始寻找正确的信息。在这种情况下，查找的结果将
会展现在方框（block）里。在这个例子中，你会看到一个红色的方框和一个黑色的方框。每个方框表示重新启动数据库服务的时间。通常在解决这个问题时，需要重新启动服务，因此甚至会看到一组或多组不同的方框，表示重启
服务发生的时间。
通过这个逻辑（logic），我可以快速判断错误何时开始的。 我知道它是在上次重启后开始的，服务重新启动两次，最后一次是服务器启动后约61分钟。


## English
As most of us know already; getting the SSPI Context error is extremely annoying and it’s infuriating that this is still happening on the latest servers.
In all most cases this is caused by the (Service Principal Name) issue. Unfortunately; while this is found based on ‘connectivity’ to a database server; the issue it’s self is only resolved using non database centric operations such as managing Service Principal Names/DNS entries on the network security layer. Once this is corrected you’ll need to restart the database service so the new SPN can be properly assigned.
To get started on finding out when this issue occurred I present to you the following SQL logic. As the image above shows; this will give you all the logs so you can review it and I’ve included a quick WHERE clause to help you get started in looking for the right bits of information. In this case you’ll get results in blocks. On this example you’ll see a red-block, and black-block. Each block represents when the database service was restarted. Typically when resolving this issue; you’ll need to restart the services so often you’ll see even one or more set of different blocks denoted by the time when the restarting service takes place.
With the logic I can quickly tell when the error started. I know it started after the previous reboot; where the services were restarted twice, and the last time was about 61 minutes after the Server came online.


---
## Logic
```SQL
use master;
set nocount on
 
--check for kerberos config
--检查kerberos config
select
    [net_transport]
,   [auth_scheme]
from
    sys.dm_exec_connections
where
    [session_id] = @@spid
 
/*********************************/
--get boot info.
--获取启动信息。
declare @start_os   datetime = (select [sqlserver_start_time] from sys.dm_os_sys_info)
declare @start_sql  datetime = (select [create_date] from sys.databases where [database_id] = 2)
 
select
    'start_os'      = left(@start_os, 19)
,   'start_sql'     = left(@start_sql, 19)
,   'notes'         = 'The SQL Service started aproximately (' + cast(datediff(minute, @start_os, @start_sql) as varchar) + ' minutes) after the OS boot.'
 
/*********************************/
--create table to list all sql files
--创建表格罗列所有的sql文档。
if object_id('tempdb..#errorlogfiles') is not null
    drop table #errorlogfiles
create table #errorlogfiles ([archive_num] int, [date] datetime, [text] nvarchar(max))
insert into #errorlogfiles exec xp_enumerrorlogs 
 
--create table to hold errors events (from files above)
--创建表格来保存错误事件（来自上面的文档）
 
if object_id('tempdb..#errorlogs') is not null
    drop table #errorlogs
create table #errorlogs ([archive_num] int null, [logdate] datetime, [processinfo] varchar(255), [text] varchar(max))
 
declare @i      int = 0
declare @maxid  int = (select max([archive_num]) from #errorlogfiles)
 
while @i <= @maxid
begin
    insert into #errorlogs ([logdate], [processinfo], [text])
    exec master..sp_readerrorlog @i, 1;
    update #errorlogs set [archive_num] = @i where [archive_num] is null;
    set @i = @i + 1;
end
 
--get error log info regarding spn events, and service start events.
--获取有关spn事件和服务启动事件的错误日志信息。
select
    'logdate'       = left([logdate], 19)
,   [processinfo]
,   [text]
from
    #errorlogs
where
    [text] like '%started%'
    or [text] like '%spn%'
order by
    [logdate] desc


```



[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

