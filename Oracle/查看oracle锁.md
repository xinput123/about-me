```
SELECT p.spid, a.serial#, c.object_name, b.session_id, b.oracle_username, b.os_user_name
FROM v$process p, v$session a, v$locked_object b, all_objects c
WHERE p.addr = a.paddr
AND a.process = b.process
AND c.object_id = b.object_id
```
