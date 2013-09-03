Programmatic Way of Generating a Column List (for inserting or selecting)
===========

When seeding databases for the first time, I sometimes find myself having to manually build a list containing all of the columns in a table.  Doing this manually normally leaves me with having to review once or twice just because I include an IDENTITY column and don’t mean to, or I add in a computed column that I didn’t realize existed / was in use.

Because of this, I have written a script to make my life easier.  Hopefully it will help you too!

Script available {{ https://github.com/awjudd/scripts/blob/master/sql/t-sql/print-csv-columns.sql }}.

-- ==============================
--  Variables to change
-- ==============================

-- The table name that we are using
DECLARE @TableFullName SYSNAME = '[Production].[Product]'
        -- Should we include computed columns in the CSV?
        , @IncludeComputedColumn BIT = 0
        -- Should we include the identity column in the CSV?
        , @IncludeIdentityColumn BIT = 1

-- Internal Variables
DECLARE @CSVInsertList NVARCHAR(MAX)
        , @ObjectID INT = OBJECT_ID(@TableFullName)

-- Make sure the object exists
IF @ObjectID IS NULL
BEGIN
    -- It doesn't, so we are done
    PRINT 'The table, ' + @TableFullName + ' does not exist.'
END
ELSE
BEGIN
    -- Grab a full list of columns that we need in CSV
    SELECT @CSVInsertList = ISNULL(@CSVInsertList + ', ', '') + c.name
    FROM sys.tables AS t
    JOIN sys.columns AS c ON t.object_id = c.object_id
    WHERE t.object_id = @ObjectID
        AND ( c.is_computed = 0 OR ( @IncludeComputedColumn = 1 AND c.is_computed = 1 ) )
        AND ( c.is_identity = 0 OR ( @IncludeIdentityColumn = 1 AND c.is_identity = 1 ) )

    -- Check if IDENTITY_INSERT will need to be on (if there is an identity column and we are including it)
    IF(EXISTS(SELECT 1 FROM sys.columns AS c WHERE c.object_id = @ObjectID AND c.is_identity = 1) AND @IncludeIdentityColumn = 1)
    BEGIN
        -- Inform the user
        PRINT 'The table has an IDENTITY column on it.'
    END

    -- Emit the CSV
    PRINT @CSVInsertList
END