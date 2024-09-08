## SQL Query Rebuild Indexes for Fragmentation
```
DECLARE @SchemaName NVARCHAR(128);
DECLARE @TableName NVARCHAR(128);
DECLARE @IndexName NVARCHAR(128);
DECLARE @RebuildSQL NVARCHAR(MAX);

DECLARE cur CURSOR FOR
SELECT 
    s.name AS SchemaName,
    OBJECT_NAME(ips.object_id) AS TableName,
    i.name AS IndexName
FROM 
    sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') ips
    JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
    JOIN sys.objects o ON ips.object_id = o.object_id
    JOIN sys.schemas s ON o.schema_id = s.schema_id
--WHERE ips.avg_fragmentation_in_percent > 90
ORDER BY 
    ips.avg_fragmentation_in_percent DESC;

OPEN cur;
FETCH NEXT FROM cur INTO @SchemaName, @TableName, @IndexName;

WHILE @@FETCH_STATUS = 0
BEGIN
    SET @RebuildSQL = 'ALTER INDEX ' + QUOTENAME(@IndexName) + ' ON ' + QUOTENAME(@SchemaName) + '.' + QUOTENAME(@TableName) + ' REBUILD;';
    EXEC sp_executesql @RebuildSQL;
    
    FETCH NEXT FROM cur INTO @SchemaName, @TableName, @IndexName;
END

CLOSE cur;
DEALLOCATE cur;
```
