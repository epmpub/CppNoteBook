# Clickhouse SQL Command



## 简介：



```SQL
SELECT COUNT(message)   from nginxdb.process_2;

SELECT message from nginxdb.access_logs;


create table t2 order by id engine=MergeTree() as (SELECT splitByChar(',', message) from nginxdb.access_logs);

TRUNCATE nginxdb.access_logs;

TRUNCATE nginxdb.process_1;
TRUNCATE nginxdb.process_2;





SELECT EventID  FROM nginxdb.access_logs_view; 


--2 Terminal--
CREATE TABLE IF NOT EXISTS  nginxdb.process_2 (
    message String
)
ENGINE = MergeTree()
ORDER BY tuple();


-- 1 Create --
CREATE TABLE nginxdb.process_1 (
    message String
)
ENGINE = MergeTree
ORDER BY tuple();




CREATE MATERIALIZED VIEW nginxdb.process_1_view
(
	RuleName String,
	UtcTime String,
	ProcessGuid String,
	ProcessId String,
	Image String,
	FileVersion String,
	Description String,
	Product String,
	Company String,
	OriginalFileName String,
	CommandLine String,
	CurrentDirectory String,
	User_ String,
	LogonGuid String,
	LogonId String,
	TerminalSessionId String,
	IntegrityLevel String,
	Hashes String,
	ParentProcessGuid String,
	ParentProcessId String,
	ParentImage String,
	ParentCommandLine String,
	ParentUser String 
	)
ENGINE = MergeTree()
ORDER BY ProcessId
POPULATE AS
WITH
--    splitByWhitespace(message) as split,
    splitByChar(',',message) as split
--    splitByRegexp('\S \d+ "([^"]*)"', message) as referer
SELECT
    split[2] AS RuleName,
    split[3] AS UtcTime,
    split[4] AS ProcessGuid,
    split[5] AS ProcessId,
    split[6] AS Image,
    split[7] AS FileVersion,
    split[8] AS Description,
    split[9] AS Product,
    split[10] AS Company,
    split[11] AS OriginalFileName,
    split[12] AS CommandLine,
    split[13] AS CurrentDirectory,
    split[14] AS User_,
    split[15] AS LogonGuid,
    split[16] AS LogonId,
    split[17] AS TerminalSessionId,
    split[18] AS IntegrityLevel,
    split[19] AS Hashes,
    split[20] AS ParentProcessGuid,
    split[21] AS ParentProcessId,
    split[22] AS ParentImage,
    split[23] AS ParentCommandLine,
    split[24] AS ParentUser
FROM
    (SELECT message FROM nginxdb.process_1);
   
    
    
    



CREATE MATERIALIZED VIEW nginxdb.access_logs_view
(
	MyDateTime String,
	EventID String,
	Message String
)
ENGINE = MergeTree()
ORDER BY MyDateTime
POPULATE AS
WITH
--    splitByWhitespace(message) as split,
    splitByChar('|',message) as split
--    splitByRegexp('\S \d+ "([^"]*)"', message) as referer
SELECT
    split[1] AS MyDateTime,
    split[2] AS EventID,
    split[3] AS Message
FROM
    (SELECT message FROM nginxdb.access_logs)
    
    

    


```



