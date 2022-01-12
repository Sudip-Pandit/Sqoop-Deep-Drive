# Sqoop-Deep-Drive

I tried to put all the real time Real Time Sqoop Commands in this repository.

Help Command
Help: This can be used to get help related to Sqoop commands.
 
Example 1: sqoop help

Example 2: sqoop help import

Example 3: sqoop help export

Example 4: sqoop help eva

===
List Database: 
The following command display all the databases present.
====
sqoop list-databases \
--connect jdbc:mysql://localhost:3306 \
--username root \
--password <>

=========
List tables:-
It displays all the tables present in "abc" database. abc can be changed with required DB name.
sqoop list-tables \
--connect jdbc:mysql://localhost:3306/demo_db \
--username root \
--password <>
  
====

EVAL: 
Eval can be used to run adhoc queries such as Select, Insert, delete & update in Database. 
demo_db can be changed with required DB name and any database query can be specified after "--query".
sqoop eval \
--connect jdbc:mysql://localhost:3306/demo_db \
--username root \
--password <> \
--driver com.mysql.cj.jdbc.Driver \
--query "SELECT * FROM order_items LIMIT 5"
  
 ====
 
How to import-all-tables?
1) The following command is used to import all the tables present in the database in one shot.
2) "--warehouse-dir" need to be used when importing all the tables.

sqoop import-all-tables \
--connect jdbc:mysql://localhost:3306/demo_db?zeroDateTimeBehavior=CONVERT_TO_NULL \
--username root \
--password <> \
--autoreset-to-one-mapper \
--warehouse-dir all_table_import

====
==> How to exclude the unused tables while importing?
In order to exclude the tables left in the database and only import the used tables, the 
following command need to use:

--exclude-tables "comma separated tablenames" can be used to exclude tables from imports.
 sqoop import-all-tables \
--connect jdbc:mysql://localhost:3306/retail_db?zeroDateTimeBehavior=CONVERT_TO_NULL \
--username root \
--password <> \
--autoreset-to-one-mapper \
--warehouse-dir all_table_import1 \
--exclude-tables "order_items_wp,orders_sqoop,orders_new,orders,orders_wp"

====
CodeGen:
 
This command can be used to generate java code which will be used by Sqoop to import/export data. 
demo_db can be changed with required DB name.
 
sqoop codegen \
--connect jdbc:mysql://localhost:3306/demo_db \
--username root \
--password <> \
--driver com.mysql.cj.jdbc.Driver \
--table order_items
 
=====
Sqoop Import Data File Formats
 => You can import the output file in the following file format types:
 
--as-textfile        This is default and will import data as plain text

--as-avrodatafile    This will import data to Avro data files

--as-parquetfile     This will Import data to Parquet files

--as-sequencefile    This will import data to SequenceFiles


====

→ Simple Import: 
In sqoop by default data is delimited by comma (,) if not specified. But we can specify the file delimited.

====
sqoop import  \
 --connect jdbc:mysql://localhost:3306/retail_db \
 --username root \
 --password <> \
 --driver com.mysql.cj.jdbc.Driver \
 --table order_items \
 --target-dir hdfs://localhost:9000/user/username/scoop_import

====

→ Import with Delimter: 
"--fields-terminated-by" can be used to specify specific delimiter for the imported data. 
In the below example, pipe(|) is used as delimiter.
 sqoop import \
 --connect jdbc:mysql://localhost:3306/retail_db \
 --username root \
 --password <> \
 --driver com.mysql.cj.jdbc.Driver \
 --table products \
 --fields-terminated-by "|" \
 --target-dir product \
 --delete-target-dir \
 --bindir $SQOOP_HOME/lib/ \
 -m 1
 
 
====

→ Import With Delete: 
"--delete-target-dir" argument can be used to delete target HDFS directory(if exists) before importing data.
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--username root \
--password <> \
--driver com.mysql.cj.jdbc.Driver \
--table products \
--target-dir product \
--delete-target-dir \
--bindir $SQOOP_HOME/lib/ \
-m 1

====
→ Import With Append: 
"--append" argument can be used to append imported data to the existing HDFS directory

sqoop import  \
--connect jdbc:mysql://localhost:3306/retail_db \
--username root \
--password <> \
--driver com.mysql.cj.jdbc.Driver \
--table orders \
--where "order_id in (11,1001,1021,1011)" \
--target-dir hdfs://localhost:9000/user/username/scoop_import/partial_column_where \
--bindir $SQOOP_HOME/lib/ \
--append

====

→ Import Without Primary Key: 
There are 2 ways to import data from tables which don't have primary keys defined.

Option 1:

sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--username root \
--password <> \
--driver com.mysql.cj.jdbc.Driver \
--table order_items_wp \
--target-dir hdfs://localhost:9000/user/username/scoop_import \
--bindir $SQOOP_HOME/lib/ \
-m 1

Option 2:

By instructing Sqoop to use particular column for breaking the data using "--split-by" argument.

sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--username root \
--password mysqlrootpassword \
--driver com.mysql.cj.jdbc.Driver \
--table order_items_wp \
--target-dir hdfs://localhost:9000/user/username/scoop_import \
--bindir $SQOOP_HOME/lib/ \
--split-by order_item_id
===============

Generic Arguments to import command

--target-dir (This is used to specify HDFS directory where data need to be imported)

--table (This is used to specify RDBMS table name from where data need to be imported)

--append (This is used to append imported data to the existing HDFS directory)

--delete-target-dir (This is used to delete target HDFS directory(if already exist) before importing data)

====

→ Import Specific columns:

"--columns" argument can be used to import specific columns.

sqoop import  \
--connect jdbc:mysql://localhost:3306/retail_db \
--username root \
--password <> \
--driver com.mysql.cj.jdbc.Driver \
--table orders \
--columns order_id,order_status,order_date \
--target-dir hdfs://localhost:9000/user/username/scoop_import/partial_column_orders \
--bindir $SQOOP_HOME/lib/

====

→ Import with Query:

" --query" attribute can be used to import with the given query.

Sqoop import with query will fail if $CONDITIONS' is not specified in Where condition. 

"--query" and "--table" attributes cannot be used together.

sqoop import  \
--connect jdbc:mysql://localhost:3306/retail_db \
--username root \
--password <> \
--driver com.mysql.cj.jdbc.Driver \
--query 'select order_status,order_id from orders WHERE $CONDITIONS' \
--target-dir hdfs://localhost:9000/user/username/scoop_import/query_orders \
--bindir $SQOOP_HOME/lib/ \
--split-by order_id

====
→ Import Data with Nulls:

If nulls are not handled properly then column with null data will be fetched and it will be import as 'null' String. 

There are different arguments to handle nulls in String and number. 

Both "--null-string" & "--null-non-string" clauses can be used in a single import.

Example 1: If nulls are not handled then it will give output like below.
 
 sqoop import  \
 --connect jdbc:mysql://localhost:3306/retail_db \
 --username root \
 --password mysqlrootpassword \
 --driver com.mysql.cj.jdbc.Driver \
 --query 'select order_status,order_id from orders where order_id in (1,100,102,101) AND $CONDITIONS' \
 --target-dir hdfs://localhost:9000/user/username/scoop_import/query_orders_null \
 --bindir $SQOOP_HOME/lib/ \
 --split-by order_id

Output: 
null,1
null,100
CLOSED,101
null,102

Example 2: 

"--null-string" argument can be used to handle NULLs in string. 

In the below example, null is being replaced with blank string.

Example 2: "--null-string" argument can be used to handle NULLs in string. In the below example, null is being replaced with blank string.
  
 sqoop import  \
 --connect jdbc:mysql://localhost:3306/retail_db \
 --username root \
 --password mysqlrootpassword \
 --driver com.mysql.cj.jdbc.Driver \
 --query 'select order_status,order_id from orders where order_id in (1,100,102,101) AND $CONDITIONS' \
 --target-dir hdfs://localhost:9000/user/username/scoop_import/query_orders_null \
 --bindir $SQOOP_HOME/lib/ \
 --split-by order_id \
 --null-string ''
 
 Example 3: 
 
 "--null-non-string" argument can be used to handle NULLs in numeric fields. In the below example, null is being replaced with -1.
  sqoop import  \
 --connect jdbc:mysql://localhost:3306/retail_db \
 --username root \
 --password mysqlrootpassword \
 --driver com.mysql.cj.jdbc.Driver \
 --query 'select order_status,order_id from orders where order_id in (1,100,102,101) AND $CONDITIONS' \
 --target-dir hdfs://localhost:9000/user/username/scoop_import/query_orders_null \
 --bindir $SQOOP_HOME/lib/ \
 --split-by order_id \
 --null-non-string -1


====

→ Import With WHERE Clause:

The same way where clasue do work in sqoop as well.

"--where" argument can be used to filter data from table during import.

In the below example, where clause is used to fetch only sep_id with values 10,20,30,40"

sqoop import  \
 --connect jdbc:mysql://localhost:3306/retail_db \
 --username root \
 --password mysqlrootpassword \
 --driver com.mysql.cj.jdbc.Driver \
 --table separtments \
 --where "dep_id in (10,20,30,40)" \
 --target-dir hdfs://localhost:9000/user/username/scoop_import/partial_column_where \
 --bindir $SQOOP_HOME/lib/ 

====

→ Import as Avro File: 

In order to import the data file format in avro, "--as-avrodatafile" argument can be used to import data in Avro data format.
 
  sqoop import  \
 --connect jdbc:mysql://localhost:3306/retail_db \
 --username root \
 --password <> \
 --driver com.mysql.cj.jdbc.Driver \
 --table order_items \
 --target-dir hdfs://localhost:9000/user/username/scoop_import/avro \
 --as-avrodatafile

====

→ Import with append:

sqoop import  \
 --connect jdbc:mysql://localhost:3306/retail_db \
 --username root \
 --password <> \
 --driver com.mysql.cj.jdbc.Driver \
 --table orders \
 --where "dep_id in (11,1001,1021,1011)" \
 --target-dir hdfs://localhost:9000/user/username/scoop_import/partial_column_where \
 --bindir $SQOOP_HOME/lib/ \
--append

====
→ Import with compression:

As we can compressed the file.

The snappy compression is good way to compress the file. 

There are many compression techniques such as Gzip, Snappy & Bzip.

Gzip: 

Compress imported data as gunzip.

All the three import examples below are same 

where " --compress", "--compression-codec Gzip" and "--compression-codec org.apache.hadoop.io.compress.GzipCodec" is used to create gunzip file.
 
 sqoop import  \
 --connect jdbc:mysql://localhost:3306/retail_db \
 --username root \
 --password mysqlroot \
 --driver com.mysql.cj.jdbc.Driver \
 --table orders \
 --target-dir hdfs://localhost:9000/user/username/scoop_import/avro_zip \
 --append \
 --split-by order_id \
 --bindir /Users/username/others/Hadoop/sqoop-1.4.7.bin__hadoop-2.6.0/lib/ \
 --compress

sqoop import  \
  --connect jdbc:mysql://localhost:3306/retail_db \
  --username root \
  --password mysqlrootpassword \
  --driver com.mysql.cj.jdbc.Driver \
  --table orders \
  --target-dir hdfs://localhost:9000/user/username/scoop_import/avro_zip \
 --append \
 --compression-codec Gzip
 
 sqoop import  \
 --connect jdbc:mysql://localhost:3306/retail_db \
 --username root \
 --password mysqlrootpassword \
 --driver com.mysql.cj.jdbc.Driver \
 --table orders \
 --target-dir hdfs://localhost:9000/user/username/scoop_import/avro_zip \
 --append \
 --compression-codec org.apache.hadoop.io.compress.GzipCodec
