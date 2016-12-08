Note: This is now directly integrated into Pydio 6.

## Introduction
Adding a full blown analytics module would be a great feature. But until then, here are a couple of SQL queries that can be performed on the database to extract some usage KPIs. They can take some time, as the “ajxp_log” table is currently not extensively indexed, for a better write-performance.

Also, of course this will not work if the Main Logger is configured to use another plugin instance (text.serial or text.syslog).

These queries where kindly contributed by Tristan Michelet. Please post yours in the comment, we’ll be glad to push them in the list!

## Files activity
### Downloads per day
    SELECT DATE(logdate) AS date, COUNT(distinct id) AS files_downloaded_per_day
    FROM ajxp_log
    WHERE severity = "INFO" AND params like "Download%"
    GROUP BY DATE(logdate);

### Most downloaded files
    SELECT SUBSTRING( params, 10 ) AS file_name, COUNT( 1 ) AS downloaded
    FROM ajxp_log
    WHERE severity = "INFO"
    AND params LIKE "Download%"
    GROUP BY file_name
    ORDER BY COUNT( 1 ) DESC
    LIMIT 10

### "# of new shared files per day"
    SELECT DATE( logdate ) AS DATE, COUNT( DISTINCT id ) AS new_shares_per_day
    FROM ajxp_log
    WHERE severity = "INFO"
    AND user
    IN (
    SELECT DISTINCT login
    FROM ajxp_user_rights
    WHERE login NOT
    IN (
    SELECT DISTINCT login
    FROM ajxp_user_rights
    WHERE repo_uuid = "ajxp.parent_user"
    )
    )
    AND params LIKE "New Share%"
    GROUP BY DATE( logdate )
    LIMIT 0 , 30

### Uploads per day
    SELECT DATE( logdate ) AS DATE, COUNT( DISTINCT id ) AS files_uploaded_per_day
    FROM ajxp_log
    WHERE user
    IN (
    SELECT DISTINCT login
    FROM ajxp_user_rights
    WHERE severity = "INFO"
    AND login NOT
    IN (
    SELECT DISTINCT login
    FROM ajxp_user_rights
    WHERE repo_uuid = "ajxp.parent_user"
    )
    )
    AND params LIKE "Upload File%"
    GROUP BY DATE( logdate )
    LIMIT 0 , 30

## Users activity
### Cumulated number of logins

    SELECT COUNT( DISTINCT id ) AS total_number_of_logins
    FROM ajxp_log
    WHERE user
    IN (
    SELECT DISTINCT login
    FROM ajxp_user_rights
    WHERE severity = "INFO"
    AND login NOT
    IN (
    SELECT DISTINCT login
    FROM ajxp_user_rights
    WHERE repo_uuid = "ajxp.parent_user"
    )
    )
    AND params LIKE "Log In%"
    LIMIT 0 , 30

### Total users connection per day

    SELECT DATE( logdate ) AS DATE, COUNT( DISTINCT id ) AS users_connections
    FROM ajxp_log
    WHERE user
    IN (
    SELECT DISTINCT login
    FROM ajxp_user_rights
    WHERE severity = "INFO"
    AND login NOT
    IN (
    SELECT DISTINCT login
    FROM ajxp_user_rights
    WHERE repo_uuid = "ajxp.parent_user"
    )
    )
    AND params LIKE "Log In%"
    GROUP BY DATE( logdate )
    LIMIT 0 , 30

### Unique users per day
    SELECT DATE( logdate ) AS DATE, COUNT( DISTINCT user ) AS users_connected
    FROM ajxp_log
    WHERE user
    IN (
    SELECT DISTINCT login
    FROM ajxp_user_rights
    WHERE severity = "INFO"
    AND login NOT
    IN (
    SELECT DISTINCT login
    FROM ajxp_user_rights
    WHERE repo_uuid = "ajxp.parent_user"
    )
    )
    GROUP BY DATE( logdate )
    LIMIT 0 , 30