# Oracle Database 23c / 23ai -- Date, Time and Globalization Enhancements

## 1. Introduction

Oracle Database 23c (also branded as **23ai**) introduces several
improvements related to:

-   Date and Time handling
-   Time zone data upgrades
-   SYSDATE and SYSTIMESTAMP behavior
-   Globalization support
-   Unicode enhancements

These improvements help developers build globally distributed
applications with accurate time handling and multilingual data support.

------------------------------------------------------------------------

# 2. Date and Time Handling Changes

## What Changed?

Oracle improved how **date and time values are managed across different
sessions, databases, and time zones**.

### Benefits

-   Consistent time handling
-   Better timezone awareness
-   Improved global application support

### Common Data Types

  Data Type                        Description
  -------------------------------- ---------------------------
  DATE                             Stores date and time
  TIMESTAMP                        Stores fractional seconds
  TIMESTAMP WITH TIME ZONE         Stores timezone info
  TIMESTAMP WITH LOCAL TIME ZONE   Normalized to DB timezone

### Example

``` sql
CREATE TABLE orders (
    order_id NUMBER,
    order_date TIMESTAMP,
    created_at TIMESTAMP WITH TIME ZONE
);
```

Insert example

``` sql
INSERT INTO orders VALUES (
1,
SYSTIMESTAMP,
SYSTIMESTAMP
);
```

------------------------------------------------------------------------

# 3. SYSDATE and SYSTIMESTAMP Improvements

## Problem Before

`SYSDATE` returned the **database server time**, which could cause
problems in distributed or container environments.

## Enhancement

New parameter:

    TIME_AT_DBTIMEZONE

This ensures consistent timestamps across sessions.

### Syntax

``` sql
ALTER SYSTEM SET TIME_AT_DBTIMEZONE = TRUE;
```

### Example

``` sql
SELECT SYSDATE FROM dual;
SELECT SYSTIMESTAMP FROM dual;
```

### Usage Scenario

Multi-region applications where the DB timezone must remain consistent.

------------------------------------------------------------------------

# 4. Enhanced Time Zone Data Upgrade

Oracle periodically updates **timezone definitions** because governments
change daylight saving rules.

### Upgrade Time Zone File

Check version

``` sql
SELECT * FROM v$timezone_file;
```

Upgrade timezone

``` sql
EXEC DBMS_DST.BEGIN_UPGRADE;
```

### Example

``` sql
SELECT CURRENT_TIMESTAMP FROM dual;
```

Result example

    20-MAR-2026 10:30:45 +05:30

------------------------------------------------------------------------

# 5. Globalization Support Improvements

Oracle improved globalization features to support:

-   Multi-language applications
-   International data formats
-   Cross-region deployments

### Important Concepts

  Feature                Description
  ---------------------- -------------------
  NLS_LANG               Language settings
  NLS_DATE_FORMAT        Date format
  NLS_TIMESTAMP_FORMAT   Timestamp format

### Example

``` sql
ALTER SESSION SET NLS_DATE_FORMAT = 'DD-MM-YYYY';
```

Query

``` sql
SELECT SYSDATE FROM dual;
```

Output

    21-03-2026

------------------------------------------------------------------------

# 6. Unicode Support Improvements

Unicode enables storing text in multiple languages.

Oracle now supports **Unicode 15.0**, enabling:

-   More language characters
-   New emojis
-   Additional scripts

### Example

``` sql
CREATE TABLE languages (
id NUMBER,
text NVARCHAR2(100)
);
```

Insert Unicode text

``` sql
INSERT INTO languages VALUES (1, 'こんにちは');
INSERT INTO languages VALUES (2, 'مرحبا');
INSERT INTO languages VALUES (3, 'Hello');
```

------------------------------------------------------------------------

# 7. Unicode IVS (Ideographic Variation Sequence)

IVS allows **different visual representations of the same character**.

Useful in languages like:

-   Japanese
-   Chinese
-   Korean

### Example

``` sql
SELECT UNISTR('\4E00') FROM dual;
```

------------------------------------------------------------------------

# 8. Unicode IVS Option

Oracle allows enabling IVS compatibility for better text rendering.

Example use cases:

-   Document management systems
-   Publishing platforms
-   Government archives

------------------------------------------------------------------------

# 9. Real World Use Cases

### Banking Systems

Correct timezone calculations for international transactions.

### E‑commerce

Order timestamps consistent across regions.

### Global Applications

Store multilingual user data.

------------------------------------------------------------------------

# 10. Summary

Key improvements in Oracle 23c / 23ai

  Area            Enhancement
  --------------- -----------------------------------
  Date & Time     Improved timezone consistency
  SYSDATE         Container aware behavior
  Timezone        Easier upgrades
  Globalization   Better NLS support
  Unicode         Unicode 15 support
  IVS             Advanced character representation

------------------------------------------------------------------------

# Conclusion

Oracle 23c / 23ai significantly improves:

-   Global database deployments
-   Multilingual data storage
-   Accurate time tracking

These enhancements are especially useful for **cloud-native and globally
distributed applications**.
