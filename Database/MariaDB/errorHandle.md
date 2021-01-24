### MySQL error code SQLSTATE code Error message

* 1011 HY000 Error on delete of '%s' (errno: %d)
* 1021 HY000 Disk full (%s); waiting for someone to free some space . . .
* 1022 23000 Can't write; duplicate key in table '%s'
1027 HY000 '%s' is locked against change
1036 HY000 Table '%s' is read only
1048 23000 Column '%s' cannot be null
1062 23000 Duplicate entry '%s' for key %d
1099 HY000 Table '%s' was locked with a READ lock and can't be updated
1100 HY000 Table '%s' was not locked with LOCK TABLES
1104 42000 The SELECT would examine more than MAX_JOIN_SIZE rows; check your WHERE and use SET SQL_BIG_SELECTS=1 or SET SQL_MAX_JOIN_SIZE=# if the SELECT is okay
1106 42000 Incorrect parameters to procedure '%s'
1114 HY000 The table '%s' is full
1150 HY000 Delayed insert thread couldn't get requested lock for table %s
1165 HY000 INSERT DELAYED can't be used with table '%s' because it is locked with LOCK TABLES
1242 21000 Subquery returns more than 1 row
1263 22004 Column set to default value; NULL supplied to NOT NULL column '%s' at row %ld
1264 22003 Out of range value adjusted for column '%s' at row %ld
1265 1000 Data truncated for column '%s' at row %ld
1312 0A000 SELECT in a stored program must have INTO
1317 70100 Query execution was interrupted
1319 42000 Undefined CONDITION: %s
1325 24000 Cursor is already open
1326 24000 Cursor is not open
1328 HY000 Incorrect number of FETCH variables
1329 2000 No data to FETCH
1336 42000 USE is not allowed in a stored program
1337 42000 Variable or condition declaration after cursor or handler declaration
1338 42000 Cursor declaration after handler declaration
1339 20000 Case not found for CASE statement
1348 HY000 Column '%s' is not updatable
1357 HY000 Can't drop a %s from within another stored routine
1358 HY000 GOTO is not allowed in a stored program handler
1362 HY000 Updating of %s row is not allowed in %s trigger
1363 HY000 There is no %s row in %s trigger
