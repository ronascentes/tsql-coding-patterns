# T-SQL Coding Patterns
A collection of amazing coding patterns in no particular order.

## Check before proceding
Explicit validation to ensure the object, variable or component is in the expected state.

### Check if a object exists
You might want to check if a object already exist before creating it. You can use this patterns for table, procedures or any object in the database.
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
Check if the current replica of a Availability Group is the primary one. First check if AG is enabled. 
```sql
IF (SELECT SERVERPROPERTY('IsHadrEnabled')) = 1
BEGIN
    IF sys.fn_hadr_is_primary_replica ( @DatabaseName ) = 1 
    BEGIN
        PRINT N'Database is not the primary';
    END;
END;

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

### Error handling
A elegant sample of using error handling in T-SQL with `TRY/ CATCH` and the explicit transactions statements: `BEGIN TRANSACTION`, `COMMIT` and `ROLLBACK`. You should set XACT_ABORT to ON to raise a run-time error so the entire transaction is terminated and rolled back.
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
### Log into errorlog

### Use the database context

## Data manipulation
### Deleting a large number of rows
It´s a script you can use to purge a large number of rows of a given table. You can start/stop the script freely, it won’t hurt anything. It´s setup to delete 1000 records per loop, but you can change this value based in your needs. The most important thing is to use a loop value which does not cause T-LOG contention.  
```sql
BEGIN  
    SET NOCOUNT, XACT_ABORT ON;
    DECLARE @ErrorMessage NVARCHAR(4000), @rows INT;  
    BEGIN TRY
        SET @rows = 1;  
  	WHILE (@rows > 0)  
  	BEGIN  
   	    BEGIN TRANSACTION  
                -- purge data older than 90 days 
                DELETE TOP(1000) FROM [dbo].[<table_name>] WHERE [LastUpdated] <= DATEADD(M, -3, GETDATE());
    		SET @rows = @@ROWCOUNT;  
            COMMIT TRANSACTION;  
        END;  
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
END;
```
