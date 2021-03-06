Stricture
===

A basic, slightly opionated data description language, inspired by Markdown.  Database engine, programming language and philosophically agnostic.

Why, you ask?  Because it felt wrong allowing some application framework to define the database structure for data models.  This simple spec can quickly spool up a data store in multiple engines, documentation and starting source code for your framework du jour.

Installation
---

You can install stricture globally and it will create a "stricture" command you can run.  These examples assume you have checked out this repository and are in the repository root folder, and have run npm install.

MicroDDL Key
---

The Stricture MicroDDL is a simple line-based database description language.  The parser (well, not really a parser, but..) will generate the model .json file that the node.js transformation and translation scripts use to generate code and documentation.

### Symbols

    !TABLE
    @Primary Numeric Identity
    %GUID
    #Number
    .Decimal
    $String [SIZE]
    *Text
    &Date
    ^Boolean

### Example:

This is a Users table, Contact table and Address table.  This file is located in the repository at "Examples/SimpleAddress.mddl"

    !User
    @IDUser
    $UserName
    $PasswordHash 42
    $FirstName 38
    $LastName 38
    $Email 60

    !Contact
    @IDContact
    #CreatingIDUser -> IDUser
    $Name 90
    $Email 60

    !Address
    @IDAddress
    #CreatingIDUser -> IDUser
    #IDContact -> IDContact
    $Address 130
    $City 48
    $State 24
    $Zip 10
    $Phone 12


### Conversion to JSON

You can translate the MicroDDL to json by running the `MicroDDL-To-JSON.sh` command:

```sh
$ node Stricture -i Examples/SimpleAddress.mddl -c Compile
Stricture JSON DDL Processing Utility
Contact: Steven Velozo <steven@velozo.com>

---


--> Running Command: Compile
--> Compiling MicroDDL to JSON
  > Input file:  Examples/SimpleAddress.mddl
  > Output file: ./build/Stricture_Output.json
  > Line #1 begins table stanza: User
  > Line #9 begins table stanza: Contact
  > Line #15 begins table stanza: Address
  > Compilation complete
```

The generated JSON in `Examples/SimpleAddress.mddl.json` looks like:
```json
{
  "Tables":
    [
      {
        "TableName": "User",
        "Columns":
        [
          {"Column":"IDUser","DataType":"ID"},
          {"Column":"UserName","DataType":"String","Size":"64"},
          {"Column":"PasswordHash","DataType":"String","Size":"42"},
          {"Column":"FirstName","DataType":"String","Size":"38"},
          {"Column":"LastName","DataType":"String","Size":"38"},
          {"Column":"Email","DataType":"String","Size":"60"}
        ]
      },
      {
        "TableName": "Contact",
        "Columns":
        [
          {"Column":"IDContact","DataType":"ID"},
          {"Column":"CreatingIDUser","DataType":"ForeignKey","Join":"IDUser"},
          {"Column":"Ordinal","DataType":"Numeric"},
          {"Column":"Name","DataType":"String","Size":"90"},
          {"Column":"Email","DataType":"String","Size":"60"}
        ]
      },
      {
        "TableName": "Address",
        "Columns":
        [
          {"Column":"IDAddress","DataType":"ID"},
          {"Column":"CreatingIDUser","DataType":"ForeignKey","Join":"IDUser"},
          {"Column":"IDContact","DataType":"ForeignKey","Join":"IDContact"},
          {"Column":"Address","DataType":"String","Size":"130"},
          {"Column":"City","DataType":"String","Size":"48"},
          {"Column":"State","DataType":"String","Size":"24"},
          {"Column":"Zip","DataType":"String","Size":"10"},
          {"Column":"Phone","DataType":"String","Size":"12"}
        ]
      }
    ]
}
```

### Diagrams

You can generate diagrams from this model.  Stricture uses the [graphviz tool chain](http://www.graphviz.org/) to generate graphs.  You must have graphviz installed and in your path to generate diagram images.

```sh
$ node Stricture -i "./build/Stricture_Output.json" -c RelationshipsFull -g -l
Stricture JSON DDL Processing Utility
Contact: Steven Velozo <steven@velozo.com>

---


--> Running Command: RelationshipsFull
Loaded graph generation file
--> Loading ./build/Stricture_Output.json
  > file loaded successfully.
--> ... creating contextual Index ==> Table lookups ...
  > Adding the table User to the lookup cache with the key IDUser
  > Adding the table Contact to the lookup cache with the key IDContact
  > Adding the table Address to the lookup cache with the key IDAddress
  > indices built successfully.
  > executing script: function
--> Building the Relationships graph...
--> ... building the connected graph DOT file ...
  > Header
  > Table Nodes
  > Connections
  > Closing
--> DOT generation complete!
--> Beginning image generation to ./build/Stricture_Output.png...
  > command: dot -Tpng ./build/Stricture_Output.dot > ./build/Stricture_Output.png
Stricture Command Execution: 12ms
  > Image generation complete
--> Loading image ./build/Stricture_Output.png in your OS.  Hopefully.
>>> Image Generation: 2ms
```

Which creates:

![Simple Table Entity Connections](https://github.com/stevenvelozo/stricture/raw/master/Examples/SimpleAddress.png)


More Complex Examples
---------------------

The infamous Northwind database has been converted to MicroDDL as an example.  It isn't 100% generating the Northwind SQL because the DDL spec doesn't cover all features used yet.

You can find it in `Examples/Northwind.mddl` ... the graph for this model is:

![Complex Table Entity Connections](https://github.com/stevenvelozo/stricture/raw/master/Examples/Northwind.png)


### MySQL

Generating MySQL Create statements is easy peasy, just run this:

```sh
$ node Stricture -i "./build/Stricture_Output.json" -c MySQL
Stricture JSON DDL Processing Utility
Contact: Steven Velozo <steven@velozo.com>

---


--> Running Command: MySQL
--> Loading ./build/Stricture_Output.json
  > file loaded successfully.
--> ... creating contextual Index ==> Table lookups ...
  > Adding the table User to the lookup cache with the key IDUser
  > Adding the table Contact to the lookup cache with the key IDContact
  > Adding the table Address to the lookup cache with the key IDAddress
  > indices built successfully.
  > executing script: function
--> Building the table create file...
  > User
  > Contact
  > Address
Stricture Command Execution: 9ms
```

Which generates some MySQL create statements in the file 'build/Stricture_Output.mysql.sql' that look like the following:

    --   [ Categories ]
    CREATE TABLE IF NOT EXISTS
        Categories
        (
            CategoryID INT UNSIGNED NOT NULL AUTO_INCREMENT,
            CategoryName CHAR(15) NOT NULL DEFAULT '',
            Description TEXT,

            PRIMARY KEY (CategoryID)
        );



    --   [ CustomerCustomerDemo ]
    CREATE TABLE IF NOT EXISTS
        CustomerCustomerDemo
        (
            CustomerID INT NOT NULL DEFAULT '0',
            CustomerTypeID INT NOT NULL DEFAULT '0'
        );

### Special Columns

The retold framework as a whole has a few concepts for each record, which provide convenience features for developers.  These are meant to maintain basic record audit tracking.

##### Create Tracking

By including either of these fields, queries will automatically stamp the user ID and create date for the record.  If they are not in your schema, nothing happens.

```
&CreateDate
#CreatingIDUser -> IDUser
```

##### Modification Tracking

By including either of these fields, queries will automatically stamp the user ID and update date for any change to the record.  If they are not in your schema, nothing happens.

```
&UpdateDate
#UpdatingIDUser -> IDUser
```


##### Deleted Bit

The deleted bit means that meadow delete operations will not actually delete the record, just flip this to true.

```
^Deleted
```


##### Delete Tracking

By including either of these fields, queries will automatically stamp the user ID and update date for deletion of the record.  If they are not in your schema, nothing happens.

```
&DeleteDate
#DeletingIDUser -> IDUser
```


##### Customer Security

If a record includes the `IDCustomer` field, it is elligible for being authorized with the `MyCustomer` authorizer.  This is handy for multi-tennant applications, where tenancy is desired for most record types by default.

### Meadow Schema Files

    You can also generate meadow schema files!  Just run stricture with the 'Meadow' command.

### Meadow Authorizers

Meadow has the capability of authorizing endpoints through built-in authorize functions, or having mix-in functions added later.  These authorizers are based on hashes.  By using an extended stricture stanza, you can define specific authorizers for endpoint/user combinations.

The roles available to meadow endpoints are:

- *Role 0*: Unauthenticated
- *Role 1*: User
- *Role 2*: Manager
- *Role 3*: Director
- *Role 4*: Executive
- *Role 5*: Administrator

The built-in authorizers include:

1. *Allow* - no matter what, allow the operation
2. *Deny* - no matter what, deny the operation
3. *Mine* - only allow me to interact with my records
4. *MyCustomer* - only allow me to interact with records that are my customer

A basic example of this might be controlling access to the inventory table:

```
[Authorization Inventory]
Read User Mine
Read Manager MyCustomer
Read Executive Deny
Read Administrator Allow
```

This will setup the security authorizers to allow Administrators to `Read` all inventory, Managers to only `Read` inventory for their customerID and Users to `Read` only inventory they have created.  As a bonus, pesky executives cannot `Read` anything.

You can also comma separate authorizers, and multiple will run.  Or even add your own hashes -- just make sure you also inject that authorizer into the meadow endpoint object.

# Change Log

* _2016-08-12_: Added the concept of Domains to models
