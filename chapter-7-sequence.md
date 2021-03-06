# Chapter 7: Sequence

`Sequence` is a `Database Object`, you can use it to generates sequential integers that organizations can use to assist with internal control or simply to serve as primary keys for tables.

## Create Sequence

```sql
CREATE SEQUENCE sequencename
    [INCREMENT BY value]
    [START WITH value]
    [{MAXVALUE value | NOMAXVALUE}]
    [{MINVALUE value | NOMINVALUE}]
    [{CYCLE | NOCYCLE}]
    [{ORDER | NOORDER}]
    [{CACHE value | NOCACHE}]
```

* The naming convention is including `_seq` at the end of the name to make it easier to identify as a sequence.
* `INCREMENT BY`: specifies the interval between two sequential values. The default value is `1`. If the sequence is incremented by a positive value, the values generated by the sequence are in ascending order. However, values generated by the sequence are in descending order when a negative value are assigned to this option.
* `START WITH`: specifies the starting value for the sequence. The default value is `1` in a ascending order, `-1` in a descending order.
* `MAXVALUE`: establish a maximum of value for the sequence. If a nagative increment is used and you set `MINVALUE`, `MAXVALUE` must also be set. Typecally, `MAXVALUE` is to set the same value as the `START WITH`
* `NOMAXVALUE`: it is default option if you do not specify `MAXVALUE`. 
  * `-1` is the highest value for a descending sequence.
  * `10^27` is the highest value for a ascending sequence.
* `MINVALUE`: establish a minimum of value for the sequence. it does not make sense when the sequence increments by a positive value.
* `NOMINVALUE`: it is default option if you do not specify `MINVALUE`.
  * `1` is the lowest value for a ascending sequence.
  * `-10^26` is the lowest value for a descending sequence.
* `CYCLE`:  reissue values from sequences after reaching the `MINVALUE` or `MAXVALUE` option.
* `NOCYCLE`: oracle server does not generate value after reaching `MINVALUE` or `MAXVALUE` option. And return an error message.
* `ORDER` and `NOORDER`: Used in application cluster environments.
  * `ORDER` option instructs oracle server to return sequence values in the same order in which requests are received.
  * `NOORDER` is the default option. However, there are no problem to generate unique value for each request.
* `CACHE`: Used to have oracle server generate a set of values ahead of time and store them in the memory. 
  * If `CACHE` is not specified, the default option of `CACHE 20` will be assigned.
  * These generated values are lost if the system crashes or users do not use the values.
* `NOCACHE`:  means each number is generated when the request is received.

```sql
CREATE SEQUENCE orders_order#_seq
    INCREMENT BY 1
    START WITH 1021
    MAXVALUE 10000
    NOCYCLE
    NOCACHE;

-- Start with -1 in descending order
CREATE SEQUENCE negative_value_seq
    INCREMENT BY -1;

-- must specify MAXVALUE option as same as START WITH while MINVALUE is assigned in a descending order.
CREATE SEQUENCE negative_value_seq
    INCREMENT BY -1
    START WITH 1
    MAXVALUE 1
    MINVALUE -100
    NOCYCLE
    CACHE 20;

-- MINVALUE is unnecessary in a ascending order.
CREATE SEQUENCE customers_customer#_seq
    START WITH 1000
    MAXVALUE 10000;

-- verify which sequences exist by querying `user_object` data dictionary.
SELECT object_name FROM user_objects WHERE object_type = 'SEQUENCE';

-- verify each setting for sequence options by query `user_sequences` data dictionary.
--      `LAST_NUMBER` column in the query result indicates the next value to be assigned in a sequence created with `NOCACHE`.
SELECT * FROM user_sequences WHERE sequence_name = UPPER('customers_customer#_seq');
```

## Using Sequence Value

`NEXTVAL`: Used to actually generate value from a sequence. In the other words, it calls the sequence object and requests the next value in a sequence.

`CURRVAL`: Used to store the current value of the sequence so that you can reference it again.

```sql
SELECT orders_order#_seq.NEXTVAL FROM dual;

SELECT orders_order#_seq.CURRVAL FROM dual;

INSERT INTO orders (order#, customer#, orderdate, shipdate, shipstreet, shipcity, shipstate, shipzip)
    VALUES (orders_order#_seq.NEXTVAL, 1010, '06-APR-09', NULL, '123 West Main', 'ATLATA', 'GA', 30418);
```

## Setting  Sequence Value

Oracle 12c allows a sequence value to be set as default value of a column.

* The sequence must be created before being referenced in the default expression of a column.
* If the sequence is dropped, the default expression referenceing the sequence must also be removed to avoid errors upon insert operations affecting the column.
* If a value is provided in the insert operation, the default sequence value will not be used.

```sql
CREATE SEQUENCE test_defval_seq
    INCREMENT BY 1
    START WITH 100,
    NOCACHE
    NOCYCLE;

CREATE TABLE test_defval(
  col1 NUMBER DEFAULT test_defval_seq.NEXTVAL,
  col2 NUMBER
);

INSERT INTO test_defval(col1, col2) VALUES (DEFAULT, 350);

INSERT INTO test_defval(col2) VALUES (351);

INSERT INTO test_defval(col1, col2) VALUES (222, 352);

INSERT INTO test_defval(col1, col2) VALUES (NULL, 353);
```

## Altering Sequence Definition

Any changes are applied to value generated after the modifications are made.

* `START WITH` can not be changed because the sequence would have to be dropped and re-created to make this change.
* The changes could not make the previously issued sequence value invalid. \(For exmaple, you can not set the `MAXVALUE` to a number less than a number that had already been generated\).

```sql
ALTER SEQUENCE sequencename
    [INCREMENT BY value]
    [{MAXVALUE value | NOMAXVALUE}]
    [{MINVALUE value | NOMINVALUE}]
    [{CYCLE | NOCYCLE}]
    [{ORDER | NOORDER}]
    [{CACHE value | NOCACHE}]


ALTER SEQUENCE orders_order#_seq
    INCREMENT BY 10;
```

## Removing Sequence

When a sequence is dropped, it does not affect any value previously generated and stored in a database table.

```sql
DROP SEQUENCE sequencename;

DROP SEQUENCE orders_order#_seq;
```

## Identity Column

An `Identity Column` is particularly suited for use as primary key column values in which random or surrogate values are needed to serve as a unique id for each row.

```sql
CREATE TABLE test_ident(
  col1 NUMBER GENERATED AS IDENTITY PRIMARY KEY,
  col2 NUMBER
);

INSERT INTO test_ident(col2) VALUES (350);
INSERT INTO test_ident(col1, col2) VALUES (DEFAULT, 351);
```

* only one `Identity Column` is allowed in one table.
* the `DEFAULT` clause can not be assigned to the same column in `CREATE TABLE` or `ALTER TABLE`.  \(An `Identity Column` automatically creates a sequence and default setting for the column\).
* A `NOT NULL` constraint and `UNIQUE` constraint are automatically applied to the column.
* An `Identity Column` must use the `NUMBER` data type.
* A value can be assigned to insert into the column.

