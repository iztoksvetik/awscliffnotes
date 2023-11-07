# Database types

## Relational

Are built to store structured data in in tables using a predefined schema. 

### Online transaction processing (OLTP)

These types of databases focus on lots of Update, Insert and Delete operations, with simple and short queries (get operations). Typically when producing *transactions*.

### Online analytical processing (OLAP)

Store historical data from OLTP and are optimised to run complex queries on that data. So few Insertions, Updates and Deletes but a lot of very complex queries (Gets). Often used in business intelligence tools. 

## Non-relational

Common denominator is lack of (explicit) schema. 

### Document

Stores semistructured and unstructured data in form of files.

### Key-value

Stores unstructured data in form of key-value pairs. 

### In-memory

Can be used for both structured and semistructured data sources.

### Graph

Purpose built to store any type of data (structured, semistructured or unstructured). 
