How to Query xEvents

## Session Definition Tables
Supported tables:
```sql
SELECT name
FROM sys.all_objects
WHERE (name LIKE 'database_%' { ESCAPE '\\' } OR 
    name LIKE 'server\_%' { ESCAPE '\\' })
AND name LIKE '%\_event%' { ESCAPE '\\' }
AND type = 'V'
ORDER BY name;
```
Gives results like:
1. database_event_session_actions
1. database_event_session_events
1. database_event_session_fields
1. database_event_session_targets
1. database_event_sessions


## Supported `action`, `event` and `target`
```sql
SELECT
    o.object_type,
    p.name         AS [package_name],
    o.name         AS [db_object_name],
    o.description  AS [db_obj_description]
FROM
    sys.dm_xe_objects  AS o
    INNER JOIN sys.dm_xe_packages AS p  ON p.guid = o.package_guid
WHERE
    o.object_type in
        (
        'action',  'event',  'target'
        )
ORDER BY
    o.object_type,
    p.name,
    o.name;
```
Gives results (top 4, long list) like:  

object_type|package_name|db_object_name|db_obj_description
---|---|---|---
action|mdmtargetpkg|mdmget_TimeStampUTC|Get time stmap UTC
action|package0|event_sequence|Collect event sequence number
action|sqlserver|client_app_name|Collect client application name
action|sqlserver|client_connection_id|Collects the optional identifier...


## Event Capture Tables
The `Dynamic Management Views` is the place where dignostic data stored.

1. sys.dm_xe_database_session_event_actions
1. sys.dm_xe_database_session_events
1. sys.dm_xe_database_session_targets
1. sys.dm_xe_database_sessions
1. sys.dm_xe_database_session_object_columns

Relations (join conditions)
```sql
SELECT
    se.name AS [session-name],
    ev.event_name,
    ac.action_name,
    st.target_name,
    se.session_source,
    st.target_data,
    CAST(st.target_data AS XML)  AS [target_data_XML]
FROM
    sys.dm_xe_database_session_event_actions AS ac

INNER JOIN sys.dm_xe_database_session_events AS ev
    ON ev.event_name = ac.event_name
    AND CAST(ev.event_session_address AS BINARY(8)) = CAST(ac.event_session_address AS BINARY(8))

INNER JOIN sys.dm_xe_database_session_object_columns AS oc
    ON CAST(oc.event_session_address AS BINARY(8)) = CAST(ac.event_session_address AS BINARY(8))

INNER JOIN sys.dm_xe_database_session_targets AS st
    ON CAST(st.event_session_address AS BINARY(8)) = CAST(ac.event_session_address AS BINARY(8))

INNER JOIN sys.dm_xe_database_sessions AS se
    ON CAST(ac.event_session_address AS BINARY(8)) = CAST(se.address AS BINARY(8))

WHERE
    oc.column_name = 'occurrence_number' ??? what
AND
    se.name        = '<session_name>'
AND
    ac.action_name = 'sql_text'
```