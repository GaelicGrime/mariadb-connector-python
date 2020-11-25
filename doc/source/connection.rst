====================
The connection class
====================

.. sectionauthor:: Georg Richter <georg@mariadb.com>

.. class:: mariadb.connection

  Handles the connection to a MariaDB or MySQL database server. It encapsulates a database session.

  Connections are created using the method :func:`~mariadb.connect()`.

-----------------------
Connection constructors 
-----------------------

.. method:: cursor(\*\*kwargs)

  Returns a new cursor object using the current connection.

  By default the result will be unbuffered, which means before executing another statement with the same connection the entire result set must be fetched.

  fetch methods of the cursor class by default return result set values as a tuple, unless named_tuple or dictionary was specified. The latter one exists for compatibility reasons and should be avoided due to possible inconsistency in case two or more fields in a result set have the same name.

  If cursor_type is set to mariadb.CURSOR_TYPE_READ_ONLY, a cursor is opened for the statement invoked with cursors execute() method.

  Supported keywords:

  - **named_tuple** (bool): When set to `True` results from fetch methods will be returned as named tuple.
  - **cursor_type** (type): When set to `CURSOR_TYPE_READ_ONLY`, server side cursor will be used.
  - **prefetch_size** (integer): The number of rows prefetched. This option will be ignored, if *cursor_type* is not `CURSOR_TYPE_READ_ONLY`
  - **buffered** (bool): When set to `True` the entire result set from a SELECT/SHOW statement will be stored in client memory
  - **prepared** (bool): When set to `True` cursor will remain in prepared state after the first :func:`~cursor.execute` method was called. Further calls to *execute()* method will ignore the sql statement.
  - **binary** (bool): When set to `True` cursor will be executed using the MariaDB client/server binary protocol.

  :return: Returns a cursor object or raises an exception if an error occurred.

.. versionadded:: 1.0.1

.. method:: xid(format_id, global_transaction_id, branch_qualifier)

  Returns a transaction ID object suitable for passing to the tpc methods of this connection

  :param format_id: Format id. If not set default value `0` will be used.
  :type format_id: integer
  :param global_transaction_id: Global transaction qualifier, which must be unique. The maximum length of the global transaction id is limited to 64 characters.
  :type global transaction_id: string
  :param branch_qualifier: Branch qualifier which represents a local transaction identifier. The maximum length of the branch qualifier is limited to 64 characters.
  :type branch_qualifier: string

------------------
Connection methods 
------------------

.. method:: commit()

  Commit any pending transaction to the database.

.. note:: 

  This method has no effect, when autocommit was set to True, or the storage engine in use doesn't support transactions.

.. method:: change_user(user, password, database)

  Changes the *user* and default *database* of the current connection.
  In order to successfully change users a valid username and password
  parameters must be provided and that user must have sufficient
  permissions to access the desired database.

  :param user: The user name for server authentication
  :type user: string
  :param password: The passoword of the user
  :type password: string
  :param database: The default database
  :type database: string

  If for any reason authorization fails an exception will be raised and the current user authentication will remain.

.. method:: close()

  Close the connection now (rather than whenever .__del__() is called).

  The connection will be unusable from this point forward; an Error
  (or subclass) exception will be raised if any operation is attempted
  with the connection. The same applies to all cursor objects trying to
  use the connection. If the connection was obtained by *ConnectionPool*,
  the connection will not be closed but returned to the pool.

.. method:: get_server_version()

  Returns numeric version of connected database server as tuple. 
  The form of the tuple is (VERSION_MAJOR, VERSION_MINOR, VERSION_PATCH).

  The get_server_version() method was added for compatibility. New applications should use the connection attribute server_version_info.

.. versionadded:: 1.0.5

.. method:: escape_string(escape_str)
 
  This function is used to create a legal SQL string that you can use in
  an SQL statement. The given string is encoded and returned as an escaped string.

  :param escape_str: The string that is to be escaped.
  :type escape_str: string

  :returns: the escaped string or NULL on error.

.. method:: kill(thread_id)

  This function is used to ask the server to terminate a database connection, specified
  by the *thread_id* parameter. 

  :param thread_id: An id which represents a database connection.
  :type thread_id: integer

.. note::
  A thread_id from other connections can be determined by executing the SQL statement ``SHOW PROCESSLIST``
  The thread_id of the current connection the current connection is stored in :data:`connection_id` attribute.

.. method:: ping()

  Checks if the connection to the database server is still available.

.. note::
  If :data:`~auto_reconnect` was set to True, an attempt will be made to reconnect to the database server in case the connection was lost

  If the connection is not available an InterfaceError will be raised.

.. method:: reconnect()

  Tries to reconnect to a server in case the connection died due to timeout
  or other errors. It uses the same credentials which were specified in
  :func:`module.connect()` method.

.. method:: reset()

  Tries to reconnect to a server in case the connection died due to timeout
  or other errors. It uses the same credentials which were specified in
  connect() method.

.. method:: rollback()

  Causes the database to roll back to the start of any pending transaction
 
  Closing a connection without committing the changes first will cause an
  implicit rollback to be performed.

 .. note::

  rollback() will not work as expected if autocommit mode was set to True or the storage engine does not support transactions.

.. method:: tpc_begin([xid])

  Begins a TPC transaction with the given transaction ID xid, which
  was created by xid() method.

  This method should be called outside of a transaction
  (i.e. nothing may have executed since the last .commit()
  or .rollback()).

  Furthermore, it is an error to call commit() or rollback() within
  the TPC transaction. A ProgrammingError is raised, if the application
  calls commit() or rollback() during an active TPC transaction.

  :param xid: A transaction id which was previously created by :func:`xid` method.
  :type xid: Dictionary

.. method:: tpc_commit([xid])

  When called with no arguments, tpc_commit() commits a TPC transaction
  previously prepared with tpc_prepare().

  If tpc_commit() is called prior to tpc_prepare(), a single phase commit
  is performed. A transaction manager may choose to do this if only a
  single resource is participating in the global transaction.

  When called with a transaction ID xid, the database commits the given
  transaction. If an invalid transaction ID is provided, a ProgrammingError
  will be raised. This form should be called outside of a transaction, and
  is intended for use in recovery.

.. method:: tpc_prepare([ xid])

  Performs the first phase of a transaction started with tpc_begin().

  A ProgrammingError will be raised if this method outside of a TPC
  transaction.

  After calling tpc_prepare(), no statements can be executed until
  :func:`~tpc_commit` or :func:`~tpc_rollback` have been called.

.. method:: tpc_recover()

  Returns a list of pending transaction IDs suitable for use with
  tpc_commit(xid) or tpc_rollback(xid).

.. method:: tpc_rollback([ xid])
 
  When called with no arguments, tpc_rollback() rolls back a TPC
  transaction. It may be called before or after :func:`tpc_prepare`.

  When called with a transaction ID xid, it rolls back the given
  transaction.

---------------------
Connection attributes
---------------------

.. data:: auto_reconnect

  Enable or disable automatic reconnection to the server if the connection
  is found to have been lost.

  When enabled, client tries to reconnect to a database server in case
  the connection to a database server died due to timeout or other errors.

.. data:: autocommit

  Toggles autocommit mode on or off for the current database connection.
   
  Autocommit mode only affects operations on transactional table types.
  Be aware that :func:`~rollback` will not work, if autocommit mode was switched on.
   
  By default autocommit mode is set to False.

.. data:: character_set

  Returns the character set used for the connection

.. data:: collation

  Returns character set collation used for the connection

.. data:: connection_id
 
  Returns the (thread) id for the current connection.

  If :data:`~auto_reconnect` was set to True, the id might change if the client reconnects to the database server

.. data:: database
 
  Returns or sets the default database for the current connection
   
  If the used database will not change, the preferred way is to specify
  the default database when establishing the connection.

.. data:: server_info
 
  Returns the alphanumeric version of connected database. The numeric version
  can be obtained via server_version() property.

.. data:: server_name

  Returns name or IP address of database server

.. data:: server_port

  Returns the database server TCP/IP port

.. data:: server_version
 
  Returns numeric version of connected database server. The form of the version
  number is VERSION_MAJOR * 10000 + VERSION_MINOR * 100 + VERSION_PATCH

.. data:: server_version_info

  Returns numeric version of connected database server as tuple. 
  The form of the tuple is (VERSION_MAJOR, VERSION_MINOR, VERSION_PATCH)

.. versionadded:: 1.0.5

.. data:: tls_cipher

  Returns TLS cipher suite in use by connection

.. data:: tls_version

  Returns TLS protocol version used by connection

.. data:: unix_socket

  Returns Unix socket name

.. data:: user

  Returns user name for the current connection

.. data:: warnings

  Returns the number of warnings from the last executed statement, or zero
  if there are no warnings.
 
.. note::

  If the sql mode ``TRADITIONAL`` is enabled an error instead of a warning will be returned. To retrieve warnings the SQL statement ``SHOW WARNINGS`` has to be used.
