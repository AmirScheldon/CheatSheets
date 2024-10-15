# Indexing Table

> optimize searching in tables by choosing a proper field.

Indexes Types:

1. Clustered index:

    > defines data orders when you want to store them in memory.There is only one Clustered index per table.Sql server choose primary key column by default.

2. Unclustered index: 

    > It does not affect the order of data storage. Creates a Pointer that store the address of certain record. You can set many Unclustered indexes per tables

3. Unique index:
    > The SQL Unique Index ensures that no two rows in the indexed columns of a table have the same values (no duplicate values allowed).\
    > HINT: Unique index do make Unique constraint

visual:\
=> object explorer => Databases => database_name => Tables => dbo.table_name => right click on indexes and select New index 
Foreign index:
> Chose a column that helps Main index to search better. forexample, Main index: first_name Foreign index: last_name

# Constraints
> SQL constraints are used to specify rules for data in a table.

constraints Types:

1. Null
2. Check: 
    > The CHECK constraint is used to limit the value range that can be placed in a column.

    visual:\
    =>object explorer => Databases => database_name => Tables => dbo.table_name => right click on Constraints and select New constraints => write you constrain in Expression field forexample, `age(field_name) >= 18` -It means no under 18 values gonna be accepted by db-

3. Unique:
    > The UNIQUE constraint ensures that all values in a column are different.\
    > HINT: A PRIMARY KEY constraint automatically has a UNIQUE constraint. However, you can have many UNIQUE constraints per table, but only one PRIMARY KEY constraint per table.

4. Primary key:
    > The PRIMARY KEY constraint uniquely identifies each record in a table.\
    >Primary keys must contain UNIQUE values, and cannot contain NULL values.\
    >A table can have only ONE primary key; and in the table, **this primary key can consist of single or multiple columns (fields)**.

    ``
    CREATE TABLE Persons (
    ID int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Age int,
    CONSTRAINT PK_Person PRIMARY KEY (ID,LastName)
    );
    ``

    Note: In the example above there is only ONE PRIMARY KEY (PK_Person). However, the VALUE of the primary key is made up of TWO COLUMNS (ID + LastName).

5. Forein Key:
    >The FOREIGN KEY constraint is used to prevent actions that would destroy links between tables.\
    >A FOREIGN KEY is a field (or collection of fields) in one table, that refers to the PRIMARY KEY in another table.\
    >The table with the foreign key is called the child table, and the table with the primary key is called the referenced or parent table.

6. Default:
    > The DEFAULT constraint is used to set a default value for a column.

    visual:\
    => Column properties => General => Default value or binding => write the value you want be inserted by default into field.

# How to add a column to a table(Query)
    ''
    ALTER TABLE table_name
    ADD column_name datatype
    ''

# How to delete a column from a table(Query)
    ''
    ALTER TABLE table_name
    DROP COLUMN column_name
    ''
# Relation between Table

1. OneToOne Relation:\
    visual:\
    => object explorer => Databases => database_name => right click on Database Diagrams => New Database Diagram => Choose databases => choose and drag a field(better to be pk) and connect it to the another field on other datebase.
    > HINT: Parent table has PK and Child table has FK.


    

    

# Select
> 
