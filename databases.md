# Database Manager

PySpigot includes a database manager that allows you to connect to and interact with SQL database types (including MySQL, MariaDB, PostgreSQL, and more) as well as MongoDB. Under the hood, PySpigot utilizes [HikariCP](https://github.com/brettwooldridge/HikariCP) for interfacing with SQL-type database servers, and the [MongoDB Java Driver](https://www.mongodb.com/docs/drivers/java-drivers/) for interfacing with MongoDB database servers.

See the [General Information](writingscripts#pyspigot39s-managers) page for instructions on how to import the database manager into your script.

This is not a comprehensive guide to working with SQL or MongoDB. Please seek out the appropriate tutorials/information if you're unsure how to use SQL or MongoDB.

## SQL

The database manager allows you to connect to and interact with SQL-type databases. 

### Database Manager Usage for SQL Databases

There are several functions available for you to use in the database manager for interacting with SQL databases. They are:

- `newHikariConfig()`: Returns a new HikariConfig object for convenience.
- `connectSql(host, port, database, username, password)`: Connects to a remote SQL database with the given host, port, database name, username, and password.
- `connectSql(host, port, database, username, password, hikariConfig)`: Connects to a remote SQL database with the given host, port, database name, username, password, using the provided HikariConfig.
- `connectSql(uri)`: Connects to a remote SQL database with the given connection string (see below for details on connection strings).
- `connectSql(uri, hikariConfig)`: Connects to a remote SQL database with the given connection string (see below for details on connection strings), using the provided HikariConfig.
- `connectSql(hikariConfig)`: Connects to a remote SQL database with the given HikariConfig.
- `disconnect(database)`: Disconnect and close an open connection to a database. Accepts the SqlDatabase object returned by `connectSql`.

All of the `connectSql` functions above accomplish the same task: they will initialize a new connection to a remote database and return an `SQLDatabase` object, which can then be used to select and update tables within the database. Which `connectSql` function you use will depend on your specific use case. Of all of the above, `connectSql(host, port, database, username, password)` is the most basic, and if you're just getting started, you should probably use this one.

The database manager also allows you to specify a [connection string/URI/URL](https://learn.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver16), if you want finer control over the connection parameters and settings. You may also specify options in the connection string. The `connectSql` functions that accept `uri` are functions that accept a connection string.

?> If you're finished using a database connection, it is good practice to close it. If you have any open database connections when your script is stopped or terminated, then these open connections will be closed automatically. If a database is closed either during or pending execution of a statement, there is no guarantee that execution of the statement will occur.

### The HikariConfig

The HikariConfig is a configuration object that allows greater control over the SQL connection. For example, it allows you to set a minimum idle time, pool size, idle timeout time, and more. For detailed information on the HikariConfig, see the [HikariCP JavaDocs](https://www.javadoc.io/doc/com.zaxxer/HikariCP/latest/com.zaxxer.hikari/com/zaxxer/hikari/HikariConfig.html).

You may also use a HikariConfig object *alone* to establish a connection (via the `connectSql(hikariConfig)` function). The HikariConfig object allows you to directly specify a host, port, database, username, and password within the object. This is probably the simplest way to establish a new connection with a remote database.

A `newHikariConfig()` function is provided in the database manager for convenience. It returns a new HikariConfig with default options that you may modify. Once you modify the options to your liking, you may then pass your HikariConfig to one of the `connectSql` functions.

### The SqlDatabase Object

Once we call any of the above `connectSql` functions, a connection is established. If the connection is established successfully, then an `SqlDatabase` object is returned. The SqlDatabase object contains the functions used to interact directly with the database:

- `getHikariDataSource()`: Returns the underlying HikariDataSource connection object.
- `select(sql)`: Executes a select statement on the database with the provided `sql`. Returns a Map (essentially the same as a python dict), with keys that represent column names and values that represent column data (a list of objects).
- `select(sql, values)`: Executes a select statement on the database with the provided `sql`, with `values` (a list of objects) that will be replaced in the `sql` statement. Returns a Map (essentially the same as a python dict), with keys that represent column names and values that represent column data (a list of objects).
- `update(sql)`: Executes an update statement on the database with the provided `sql`. Returns an int that signals the number of rows affected by the update.
- `update(sql, values)`: Executes an update statement on the database, with `values` (a list of objects) that will be replaced in the `sql` statement. Returns an int that signals the number of rows affected by the update.

`sql` in the above functions is simply an SQL statement. For example:
```sql
SELECT * FROM test_table;
```

We can also define values that will be automatically inserted into the SQL statement. For example, we can define an SQL statement like the following:
```sql
SELECT * FROM test_table WHERE id = ?;
```
This statement is called a **parameterized query**, because it contains question marks (`?`). Question marks in the SQL statement function as *placeholders*, or, values that will be inserted later. This is where the `values` parameter comes into play: question marks are replaced with the values that are passed in `values`. For example, if we call
```python
sql.select('SELECT * FROM test_table WHERE id = ?', [10])
```
then the *actual* SQL statement sent to the server will be
```sql
SELECT * FROM test_table WHERE id = 10;
```
because the question mark was replaced with the value `10` we passed in `values`. This also works if we want to pass multiple values in `values`:
```python
sql.update('INSERT INTO test_table (id, val) VALUES (?, ?)', [11, 1])
```
The above statement effectively becomes `INSERT INTO test_table (id, val) VALUES (11, 1);`. Note that the ordering of objects in `values` is important as it determines the order in which the placeholders are replaced in the SQL statement. The first question mark is replaced with the object at position zero, the second question mark is replaced with the object at position one, and so on.

### Code Example for SQL Databases

The following code connects to and performs some simple operations on a remote SQL database. The table being interacted with is named `test_table` and has columns `id` (auto-increment, not null, unique) and `value` (not null):

```python
import pyspigot as ps

db_manager = ps.database_manager()

sql = db_manager.connectSql('localhost', '3306', 'test', 'root', '')

data = sql.select('SELECT * FROM test_table ORDER BY val DESC;')

for column, col_data in data.items():
    print(column)
    print(col_data)

rows_affected = sql.update('INSERT INTO test_table (id, val) VALUES (?, ?)', [11, 1])
print('Rows affected: ' + str(rows_affected))

db_manager.disconnect(sql)
```

On line 1, we import PySpigot as `ps` to utilize the database manager.

On line 3, we get the database manager from `ps` and set it to `db_manager`.

On line 5, we establish a new connection with `connectSql` and assign the returned object to `sql`. This variable is what we will subsequently use to interact with the database.

On line 7, we select all data from `test_table` and order it by the column `val`, descending, and assign the selected data to the `data` variable.

On lines 9-11, we loop through all of the data and print the column name along with the data corresponding to that column.

On line 13, we execute an update on `test_table` and insert a new row with values 11 for `id` and 1 for `val`, and assign the result (number of rows affected) to the variable `rows_affected`.

On line 14, we print the number of rows affected.

On line 16, we close the database connection by calling the `disconnect` function from the database manager and passing our database object we got earlier when we connected with `connectSql`.

!> It's important to keep in mind that interacting with any remote database is an *I/O operation*. If we run this code on the main server thread (I.E., not in an asynchronous task), then the server *will hang* and be unresponsive until the interaction with the remove server completes. For laggy and/or high-latency connections, this can cause significant amounts of server lag. To avoid lag, it is best practice to wrap all *I/O operations* in an asynchronous task. See the section below for an example.

### Code Example for SQL Databases, Asynchronous

The following code takes the above code example but wraps the interaction with the database in an asynchronous task. Of note, the connection to the database (`connectSql` function on line 6) is still synchronous, but we want this to remain synchronous because a failed connection affects the rest of the script's execution, and the script should terminate if the connection failes (I.E. an error is thrown).

The code below makes use of the task manager's callback task, which runs a task asychronously, then calls back to the main server thread when the asychronous task finishes via another user-defined function.

```python
import pyspigot as ps

task_manager = ps.task_manager()
db_manager = ps.database_manager()

sql = db_manager.connectSql('localhost', '3306', 'test', 'root', '')

def select_data():
    data = sql.select('SELECT * FROM test_table ORDER BY val DESC;')
    return data

def sync_callback(data):
    for column, col_data in data.items():
        print(column)
        print(col_data)

task_manager.runSyncCallbackTask(select_data, sync_callback)

def update():
	rows_affected = sql.update('INSERT INTO test_table (id, val) VALUES (?, ?)', [11, 1])
	print('Rows affected: ' + str(rows_affected))

task_manager.runTaskAsync(update)
```

## MongoDB

The database manager allows you to connect to and interact with Mongo databases (MongoDB).

### Database Manager Usage for MongoDB

There are several functions available for you to use in the database manager for interacting with MongoDB. They are:

- `newMongoClientSettings()`: Returns a new MongoClientSettings builder for convenience.
- `connectMongo(host, port, username, password)`: Connects to a remote MongoDB server with the given host, port, username, and password.
- `connectMongo(host, port, username, password, clientSettings)`: Connects to a remote MongoDB server with the given host, port, username, and password, using the given MongoClientSettings.
- `connectMongo(uri)`: Connects to a remote MongoDB server with the given connection string.
- `connectMongo(uri, clientSettings)`: Connects to a remote MongoDB server with the given connection string, using the given MongoClientSettings.
- `connectMongo(clientSettings)`: Connects to a remote MongoDB server with the given MongoClientSettings.
- `disconnect(database)`: Disconnect and close an open connection to a database. Accepts the MongoDatabase object returned by `connectMongo`.

The database manager allows you to specify a [connection string/URI/URL](https://learn.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver16), if you want finer control over the connection parameters and settings. You may also specify options in the connection string. The `connectMongo` functions that accept `uri` are functions that accept a connection string.

These function very similar to the functions for initializing a connection to an SQL database. The only real difference is that instead of a HikariConfig, we can use a MongoCLientSettings to specify connection settings.

?> If you're finished using a database connection, it is good practice to close it. If you have any open database connections when your script is stopped or terminated, then these open connections will be closed automatically. If a database is closed either during or pending execution of an operation, there is no guarantee that execution of the opteration will occur.

### The MongoClientSettings

The MongoClientSettings is a configuration object that allows greater control over the MongoDB connection. For example, it allows you to set connection pool settings, connection string, encryption settings, read preference, write preference, etc. For detailed information on the MongoClientSettings, see the [MongoDB documentation](https://www.mongodb.com/docs/drivers/java/sync/current/fundamentals/connection/mongoclientsettings/).

You may also use a MongoClientSettings object *alone* to establish a connection (via the `connectMongo(clientSettings)` function). The MongoClientSettings builder object allows you to directly specify a connection string or host, port, database, username, and password within the object. This is probably the simplest way to establish a new connection with a remote database.

A `newMongoClientSettings()` function is provided in the database manager for convenience. It returns a new MongoClientSettings builder with default options that you may modify. Once you modify the options to your liking, you may then call `build()` on the object and pass the result to one of the `connectMongo` functions.

!> Once you finish setting the options you'd like to set for the MongoClientSettings, you must call `build()` to build the settings into a readable object prior to passing it to one of the `connectMongo` functions.

### The MongoDatabase Object

Once we call any of the above `connectMongo` functions, a connection is established. If the connection is established successfully, then a `MongoDatabase` object is returned. The MongoDatabase object contains the functions used to interact directly with the database.

There are too many functions in the MongoDatabase object to list them all here. For a complete list, see the [PySpigot JavaDocs](https://javadocs.magicmq.dev/pyspigot/dev/magicmq/pyspigot/manager/database/mongo/MongoDatabase.html).

Additionally, the [MongoDB documentation](https://www.mongodb.com/docs/drivers/java/sync/current/usage-examples/) contains detailed documentation and example code. I highly recommend you check it out for information on how to work with the MongoDatabase object, since many of the functions I have defined in MongoDatabase are simply passthroughs to the underlying MongoDB library.

?> If you're finished using a database connection, it is good practice to close it. If you have any open database connections when your script is stopped or terminated, then these open connections will be closed automatically. If a database is closed either during or pending execution of an operation, there is no guarantee that execution of the operation will occur.

### Code Example for MongoDB

The following code established a connection to a remote MongoDB server, creates a collection, inserts a document, and then retrieves the document that was created.

```python
import pyspigot as ps

db_manager = ps.database_manager()

mongo = db_manager.connectMongo('localhost', '27017', None, None)

if (not mongo.doesCollectionExist('test', 'test_collection')):
    mongo.createCollection('test', 'test_collection')

mongo.insertDocument('test', 'test_collection', mongo.createDocument('test_key', 'test_value'))

documents = mongo.getDocuments('test', 'test_collection')

for document in documents:
    print(document)
```

!> The above code is run sycnchronously. As outlined above, it is best to interact with an external database server in an asychronous context to avoid server lag.

## To summarize: {docsify-ignore}

- The DatabaseManager allows you to connect to and interact with SQL-type and Mongo databases.
- Use the `connectSql` functions (along with the provided arguments, based on your specific situation) to connect to an SQL database.
- Use the `connectMongo` functions (along with the provided arguments, based on your specific situation) to connect to a Mongo database.
- When connecting to a database, a database object (either `SqlDatabase` or `MongoDatabase`) is returned by the connect function, which is then used to interact with the database.
- Interacting with a database is an *I/O operation*, and these interactions should be done in an asynchronous context, such as within a scheduled asynchronous task or within a callback task.
- Database connections are closed automatically when a script is stopped. At any other time, if you are finished using a database, it should be closed by calling the database manager `disconnect` function. The `disconnect` function takes the database object that was procured when connecting to the database.