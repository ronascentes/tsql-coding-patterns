# T-SQL Coding Patterns
A collection of amazing coding patterns in no particular order.

## Check before proceding
Explicity validation to ensure the object, variable or component is in the correct state.

### Check if a object exists
You might want to check if a object already exist before creating it. You can use this patterns for table, procedures or any object in the database
```sql
IF EXISTS (SELECT [object_id] FROM tempdb.sys.objects (NOLOCK) WHERE [object_id] = OBJECT_ID('tempdb.dbo.#<table_name>'))      
	DROP TABLE #<table_name>;
```

### Check if database exists
Does the database exists and is it up and running?
```sql
IF EXISTS (SELECT CASE WHEN DATABASEPROPERTYEX(name, 'Status') = 'ONLINE' THEN 1 END FROM master.dbo.sysdatabases WHERE name = <db_name>)
	PRINT N'<db_name> is available';
```
### Check if is primary node
Check if the current node of a cluster is the primary replica of the Availability Group
```sql
IF sys.fn_hadr_is_primary_replica ('<db_name>') = 
```

### Null or empty
A common pattern to check if a varible´s content is valid.
```sql
DECLARE @Var VARCHAR(255) = '';
IF (@Var IS NULL OR @Var = '')
	PRINT N'@Var is null or empty';
```

## General programming
General programming practices and patterns

### New line

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
