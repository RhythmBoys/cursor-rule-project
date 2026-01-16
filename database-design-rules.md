# Database Design Rules

## Naming Conventions

### Table Naming Rules
1. **Use plural form**: Table names should use plural form, e.g., `users`, `orders`, `products`
2. **Use lowercase letters and underscores**: Table names should be all lowercase, with words separated by underscores, e.g., `user_profiles`, `order_items`
3. **Avoid abbreviations**: Unless it's a widely recognized abbreviation (e.g., `id`, `url`, `api`), use full words
4. **Clear table names**: Table names should clearly describe the data they store

### Column Naming Rules
1. **Use lowercase letters and underscores**: Column names should be all lowercase, with words separated by underscores, e.g., `user_id`, `created_at`, `is_active`
2. **Primary key naming**: Primary keys should be uniformly named `id`, with type `BIGINT` or `INTEGER`, auto-increment
3. **Foreign key naming**: Foreign key columns should follow the format `{referenced_table_name}_id`, e.g., `user_id`, `order_id`
4. **Boolean field naming**: Boolean fields should use `is_` or `has_` prefix, e.g., `is_active`, `is_deleted`, `has_permission`
5. **Timestamp field naming**:
   - Creation time: `created_at` (TIMESTAMP)
   - Update time: `updated_at` (TIMESTAMP)
   - Deletion time: `deleted_at` (TIMESTAMP, for soft delete)
   - Created by: `created_by` (BIGINT, references user ID)
   - Updated by: `updated_by` (BIGINT, references user ID)
6. **Status field naming**: Status fields should use `status` or `{business_meaning}_status`, e.g., `status`, `order_status`
7. **Avoid reserved words**: Do not use database reserved words as column names (e.g., `order`, `group`, `user`)

### Index Naming Rules
1. **Primary key index**: `pk_{table_name}`, e.g., `pk_users`
2. **Unique index**: `uk_{table_name}_{column_name}`, e.g., `uk_users_email`
3. **Regular index**: `idx_{table_name}_{column_name}`, e.g., `idx_users_created_at`
4. **Composite index**: `idx_{table_name}_{column1}_{column2}`, e.g., `idx_orders_user_id_status`

### Constraint Naming Rules
1. **Foreign key constraint**: `fk_{table_name}_{referenced_table_name}`, e.g., `fk_orders_users`
2. **Check constraint**: `ck_{table_name}_{column_name}`, e.g., `ck_users_age`
3. **Unique constraint**: Use unique index naming rules

## Field Type Specifications

### Numeric Types
- **Primary key ID**: `BIGINT` or `INTEGER` (choose based on data volume)
- **Integers**: Choose `TINYINT`, `SMALLINT`, `INTEGER`, `BIGINT` based on value range
- **Decimals**: Use `DECIMAL(precision, scale)` or `NUMERIC(precision, scale)`
- **Floating point**: Use `FLOAT` and `DOUBLE` with caution, prefer `DECIMAL`

### String Types
- **Short strings** (< 255 characters): `VARCHAR(length)`, must specify length
- **Long text**: `TEXT` or `LONGTEXT`
- **Fixed length**: `CHAR(length)`, only for fixed-length strings (e.g., status codes, enum values)

### Date and Time Types
- **Date**: `DATE`
- **DateTime**: `TIMESTAMP` or `DATETIME`
  - `TIMESTAMP`: Automatically handles timezone, smaller range
  - `DATETIME`: Does not handle timezone, larger range
- **Timestamp**: Use `TIMESTAMP` to store Unix timestamps

### Boolean Types
- Use `TINYINT(1)` or `BOOLEAN` (MySQL)
- Use `BOOLEAN` (PostgreSQL)
- Values: `1`/`0` or `true`/`false`

### JSON Types
- Use `JSON` or `JSONB` (PostgreSQL) to store structured data
- Avoid excessive use of JSON, prefer relational design

## Field Design Specifications

### Required Fields
1. **Primary key**: Every table must have a primary key
2. **Timestamps**: It is recommended that every table includes `created_at` and `updated_at`
3. **Business required fields**: Set `NOT NULL` based on business requirements

### Default Values
1. **Timestamp fields**: Set default value to current time for `created_at` and `updated_at`
2. **Boolean fields**: Set reasonable default values, e.g., `is_active` defaults to `true` or `false`
3. **Status fields**: Set initial status default values

### Field Length
1. **String fields**: Set reasonable length based on actual business requirements, avoid too long or too short
2. **Reserve expansion space**: For fields that may expand, reserve appropriate space

### Comments
1. **Table comments**: Every table should have comments explaining its purpose
2. **Column comments**: Important columns should have comments explaining their meaning and value ranges
3. **Index comments**: Complex indexes should have comments explaining their purpose

## Table Design Specifications

### Table Structure
1. **Column order**:
   - Primary key `id`
   - Business columns (sorted by importance)
   - Foreign key columns
   - Timestamp columns (`created_at`, `updated_at`, `deleted_at`)
   - Operator columns (`created_by`, `updated_by`)
2. **Avoid redundancy**: Follow database normalization, avoid data redundancy
3. **Moderate denormalization**: In scenarios with high performance requirements, moderate denormalization is acceptable

### Relationship Design
1. **One-to-one relationship**: Use foreign key association, or merge into the same table
2. **One-to-many relationship**: Add foreign key on the many side
3. **Many-to-many relationship**: Use intermediate table, naming format `{table1}_{table2}`, e.g., `user_roles`

### Soft Delete
1. **Use `deleted_at` field**: Implement soft delete, type `TIMESTAMP NULL`
2. **Index**: Create index for `deleted_at` field to improve query efficiency
3. **Unique constraint**: If using soft delete, unique constraints need to consider `deleted_at IS NULL` case

## Index Design Specifications

### Index Principles
1. **Primary key auto-index**: Primary keys automatically create indexes
2. **Foreign key index**: All foreign key columns should create indexes
3. **Query field index**: Create indexes for fields frequently used in WHERE, JOIN, ORDER BY
4. **Avoid excessive indexing**: Indexes reduce write performance, do not create unnecessary indexes

### Composite Index
1. **Leftmost prefix principle**: Composite indexes follow the leftmost prefix principle
2. **Column order**: Place columns with high selectivity first
3. **Covering index**: Try to create covering indexes to reduce table lookups

## Examples

### Table Design Example
```sql
-- Users table
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT 'User ID',
    username VARCHAR(50) NOT NULL COMMENT 'Username',
    email VARCHAR(100) NOT NULL COMMENT 'Email',
    password_hash VARCHAR(255) NOT NULL COMMENT 'Password hash',
    is_active TINYINT(1) DEFAULT 1 COMMENT 'Is active',
    status VARCHAR(20) DEFAULT 'pending' COMMENT 'Status',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT 'Created at',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'Updated at',
    deleted_at TIMESTAMP NULL COMMENT 'Deleted at',
    created_by BIGINT NULL COMMENT 'Created by user ID',
    updated_by BIGINT NULL COMMENT 'Updated by user ID',
    
    UNIQUE KEY uk_users_email (email),
    UNIQUE KEY uk_users_username (username),
    INDEX idx_users_status (status),
    INDEX idx_users_created_at (created_at),
    INDEX idx_users_deleted_at (deleted_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Users table';

-- Orders table
CREATE TABLE orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT 'Order ID',
    user_id BIGINT NOT NULL COMMENT 'User ID',
    order_no VARCHAR(32) NOT NULL COMMENT 'Order number',
    total_amount DECIMAL(10, 2) NOT NULL COMMENT 'Total amount',
    order_status VARCHAR(20) DEFAULT 'pending' COMMENT 'Order status',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT 'Created at',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT 'Updated at',
    deleted_at TIMESTAMP NULL COMMENT 'Deleted at',
    created_by BIGINT NULL COMMENT 'Created by user ID',
    updated_by BIGINT NULL COMMENT 'Updated by user ID',
    
    UNIQUE KEY uk_orders_order_no (order_no),
    INDEX idx_orders_user_id (user_id),
    INDEX idx_orders_status (order_status),
    INDEX idx_orders_created_at (created_at),
    FOREIGN KEY fk_orders_users (user_id) REFERENCES users(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='Orders table';
```

## Notes

1. **Character set**: Uniformly use `utf8mb4` character set, supporting full UTF-8 encoding (including emoji)
2. **Storage engine**: MySQL recommends using `InnoDB`, supporting transactions and foreign keys
3. **Timezone handling**: Clarify timezone settings, recommend storing in UTC, convert at application layer
4. **Version control**: Use version control for database changes (e.g., Flyway, Liquibase)
5. **Performance considerations**: Consider partitioning for large tables, caching for hot data

