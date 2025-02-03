# T-SQL Coding Patterns
A collection of amazing coding patterns in no particular order.

## Check before proceding

You might want to check if a object already exist before creating it. You can use this patterns for table, procedures or any object in the database
```sql
IF EXISTS (SELECT [object_id] FROM tempdb.sys.objects (NOLOCK) WHERE [object_id] = OBJECT_ID('tempdb.dbo.#searchdata'))      
	DROP TABLE #searchdata;
```

Does the database exists and is it up and running?
```sql
IF EXISTS (SELECT CASE WHEN DATABASEPROPERTYEX(name, 'Status') = 'ONLINE' THEN 1 END FROM master.dbo.sysdatabases WHERE name = <db_name>)
	PRINT N'<db_name> is available';
```

Check if the current node of a cluster is the primary replica of the AG
```sql
IF sys.fn_hadr_is_primary_replica ('<db_name>') = 
```

Null or Empty, a common pattern to check if a varible´s content is valid.
```sql
DECLARE @Var VARCHAR(255) = '';
IF (@Var IS NULL OR @Var = '')
	PRINT N'@Var is null or empty';
```

## General programming
New line
```sql
DECLARE @NewLineChar AS CHAR(2) = CHAR(13) + CHAR(10);
```

Error handling
```sql
SET NOCOUNT, XACT_ABORT ON;  
DECLARE @ErrorMessage NVARCHAR(4000), @rows INT;  
BEGIN TRY  
	BEGIN TRANSACTION
		-- put your SQL statement here
		-- ...
		-- ...
	COMMIT TRANSACTION;  
END TRY  
BEGIN CATCH  
	SELECT @ErrorMessage = ERROR_MESSAGE();   
	PRINT @ErrorMessage;  
	IF (XACT_STATE()) = -1  
		ROLLBACK;  
	IF (XACT_STATE()) = 1  
		COMMIT;   
	THROW;  
END CATCH;
```
