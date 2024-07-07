# Database Field Rules

This guide specifies rules for certain fields in the database schema.

## Fields

### 1. `created_by`
- **Type:** `VARCHAR(36)`
- **Default Value:** `'system'`
- **Description:** This field records the identifier of the user who created the record. If no user is specified, the default value `'system'` will be used.

```xml
    <column name="created_by" type="VARCHAR(36)" defaultValue="system">
        <constraints nullable="false"/>
    </column>
```

### 2. `created_date`
- **Type:** `TIMESTAMP`
- **Default Value:** `NOW()`
- **Description:** This field records the date and time when the record was created. It will automatically be set to the current date and time when the record is created.
- **Note:** This field should not be manually updated; it is intended to capture the creation timestamp only.
```xml
    <column name="created_date" type="TIMESTAMP" defaultValueComputed="now()">
        <constraints nullable="false"/>
    </column>
```

### 3. `display_order`
- **Type:** `INTEGER`
- **Default Value:** `AUTO_INCREMENT`
- **Description:** This field determines the order in which records are displayed. It automatically increments for each new record, ensuring a unique and sequential order value.
- **Note:** This field is intended for sorting and display purposes. Manual updates to this field should be done with caution to maintain order integrity.

```xml
    <column name="display_order" type="INTEGER" autoIncrement="true">
        <constraints nullable="false"/>
    </column>
```

