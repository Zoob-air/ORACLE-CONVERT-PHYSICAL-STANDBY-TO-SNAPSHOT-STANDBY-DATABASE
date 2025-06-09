# Convert Physical Standby to Snapshot Standby Database (Oracle)

This document provides step-by-step instructions to convert a **physical standby database** to a **snapshot standby database**, and later revert it back to physical standby. This is useful for performing temporary read-write operations on the standby database for testing or other purposes.

---

## Prerequisites

* Ensure the standby database is in MOUNT state.
* Flashback Database must be enabled.
* Sufficient space must be allocated for `db_recovery_file_dest`.

---

## Step 1: Check Current Database Role and Mode

```sql
SELECT name, open_mode, database_role FROM v$database;
```

* Confirms if the database is currently a **physical standby** and its state.

---

## Step 2: Cancel Redo Apply

```sql
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE CANCEL;
```

* Stops the managed recovery process which is required before the conversion.

---

## Step 3: Enable Flashback Database

```sql
SHUTDOWN IMMEDIATE;
STARTUP MOUNT;
SHOW PARAMETER db_recovery_file_dest;
ALTER SYSTEM SET db_recovery_file_dest_size=1G;
ALTER SYSTEM SET db_recovery_file_dest='/u03/testdb/TESTER1';
ALTER DATABASE FLASHBACK ON;
SELECT flashback_on FROM v$database;
```

* **Shutdown and startup in MOUNT mode** to make configuration changes.
* **Set flashback location and size**.
* **Enable Flashback Database** to allow future reversion.

---

## Step 4: Convert to Snapshot Standby

```sql
SELECT status FROM v$instance;
ALTER DATABASE CONVERT TO SNAPSHOT STANDBY;
ALTER DATABASE OPEN;
SELECT database_role FROM v$database;
SELECT name, open_mode, database_role FROM v$database;
SELECT NAME, GUARANTEE_FLASHBACK_DATABASE FROM v$restore_point;
```

* **Convert** the standby to snapshot standby.
* **Open the database** in read-write mode.
* **Check restore points** to confirm flashback support.

---

## Step 5: Revert to Physical Standby

```sql
SHUTDOWN IMMEDIATE;
STARTUP MOUNT;
SELECT FLASHBACK_ON FROM v$database;
ALTER DATABASE CONVERT TO PHYSICAL STANDBY;
SELECT status FROM v$instance;
SELECT name, open_mode, database_role FROM v$database;
```

* **Shutdown and startup in MOUNT** again.
* **Convert back** to physical standby.
* **Validate** that the role is restored properly.

---

## Step 6: Resume Redo Apply

```sql
SHUTDOWN IMMEDIATE;
STARTUP;
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE USING CURRENT LOGFILE DISCONNECT;
SELECT name, open_mode, database_role FROM v$database;
```

* Final step to **resume redo apply** and maintain sync with primary.

---

## Notes

* Make sure you have enough disk space in the FRA (Fast Recovery Area).
* Flashback must be kept enabled as long as the database is in snapshot standby.
* Reversion to physical standby automatically rolls back to the guaranteed restore point.

---

> For further reading: [Oracle Documentation - Data Guard Concepts and Administration](https://dbaclass.com/article/convert-physical-standby-to-snapshot-standby-database/#google_vignette)
