There are 2 sources of boilerplate code:

1. cursor.execute() and cursor.fetchall()/cursor.rowcount/cursor.lastrowid have to be separately called to execute a SQL and then acquire the results/information of the execution;
1. connection/cursor objects have to be acquired first, and then closed. The transaction has to be committed or aborted each time.

The first problem can be easily fixed by wrapping your own function around the execute() and fetchall() calls. At the lowest level, the DBI doesn't want to incur the cost of info fetching on those callers who don't care about the info. This is easily understandable by one from the C++ background (ie, me). However, it's unwieldy, and at a higher level, I can make my own decision to pay the price each time I execute a SQL statement.

The second source of boilerplate code comes from the execution model of the Python DBI. The DBI is an API to be called, rather than a framework to make the calls. When there is no transaction to commit, the passive API/library model works fine and unobtrusive. However, so long as transactions are involved, boilerplate code emerges as all transactions are alike. The dbtxn library manages the transaction, setting up the necessary exception-safe resource releasing mechanism.

The dbtxn library does more than simply managing transactions. Actually, it separates the business logic which doesn't care about the underlying database from the callers who know and play with the dirty details. The business logic only needs to know about the SQLs and the processing of the result set, but not the connection, cursor or transaction involved. And business logic should be composable into bigger transactions, without having to pass cursors around. This is all possible with dbtxn.

Generators instead of plain functions are chosen as the interface of the library. This is straightforward, as the callee of the dbtxn() function _generates_ SQLs to be executed by dbtxn(), which forms an obvious producer/consumer relationship. You can yield multiple SQLs from the same generator without returning to dbtxn(), and without dbtxn() passing in anything when calling the logic, polluting its interface. dbtxn() also handles nested generators, making the SQL generators composable. SQLs in the same transaction can be grouped into multiple generators, making the code more maintainable.

Using generators makes mocking the database for the business logic a breeze. You don't need a database or mocking the DBI. You don't even need the dbtxn library. To test the logic, you just need to call it and send() in data as the result set. Also, the send() calls and the _yield_s can be easily paired, without any bookkeeping.

Also, `@in_txn` decorator is provided to remove the cluttering calls of dbtxn(). Decorated by it, the generators can be called as if they're ordinary functions. `@for_recurse` is provided to make nested generator look like ordinary functions.

In retrospect, dbtxn() looks a lot like a monad, namely, the STM monad. The goal to encapsulate computation is close, but the implementation is not. I will work to make it more monadic.
