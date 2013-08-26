Programmatically Determine the last time an object was modified
===========

Every once in a while, I get an email saying that something must have changed on the live site because there is a new issue that "just popped up".

At those times, in order to verify that nothing has actually changed recently, I run the following query.

    SELECT FullName = SCHEMA_NAME(schema_id) + '.' + name, create_date, modify_date
    FROM sys.objects 
    ORDER BY modify_date DESC, create_date DESC

This tells you the last time that an object in the database has changed.

Hopefully this will help you to quickly determine if an object has changed recently, or if it was a pre-existing bug.