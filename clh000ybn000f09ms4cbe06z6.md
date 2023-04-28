---
title: "Managing Dataflow and SQL Database with Concurrency Control"
seoTitle: "Concurrency Control: Dataflow & SQL Management"
seoDescription: "Optimize SQL databases with concurrency control to prevent data inconsistencies and conflicts in high workload applications."
datePublished: Fri Apr 28 2023 03:33:10 GMT+0000 (Coordinated Universal Time)
cuid: clh000ybn000f09ms4cbe06z6
slug: managing-dataflow-and-sql-database-with-concurrency-control
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/lRoX0shwjUQ/upload/67a4c3bcc547f149b4ebb69a1ed13e98.jpeg
tags: postgresql, databases, concurrency, sql, system-design

---

Concurrency control is a crucial aspect of developing applications that can handle multiple user requests simultaneously. With SQL databases, it is not uncommon for multiple users to access the same data at the same time. This can lead to problems such as data inconsistencies and conflicts. To prevent such issues from occurring, software engineers must implement concurrency control techniques when querying SQL databases. In this article, we will delve deeper into the importance of concurrency control in SQL databases and explore some of the commonly used techniques to achieve it.

# Problem

When you're developing an application that needs to handle a high workload or requires high availability, one of the first things you might consider is running it on multiple servers. This is a common approach that helps to ensure that your application can handle a large number of user requests and maintain its performance and uptime. By distributing the workload across multiple servers, you can also reduce the risk of a single point of failure, which can help to keep your application running smoothly.

But that's not all. You may also have a sequence of steps that only need to be processed once across the system. Duplicating these steps can cause your system to execute the wrong business logic, especially when interacting with a database.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682652456569/5c6d258c-fbdb-4da7-aa42-3402d6b3c6c1.png align="center")

Simple case of duplicate behavior

We can list several common scenarios that result in this problem.

* You can build an inventory where your items can be marked as either available for sale or sold. However, how can you ensure that two customers can't buy the same item at the same time? In technical terms, this is known as a race condition, where two or more requests arrive and are processed simultaneously.
    
* You have multiple indexers for high availability, which subscribe to the blockchain and listen for specific types of events. For example, when an "event index" occurs, you only want to store the event once. Similarly, when an "event update" occurs, you only want to trigger the update once.
    

# Theoretical basis

To address the issue at hand, implementing database locking or a single flow data streaming approach can help to deduplicate a bulk of data. These solutions can prevent conflicts and ensure efficient handling and processing of data.

## Database Locking

Database locking is one of the most common mechanisms that helps us achieve concurrency control in a database by preventing multiple transactions from accessing the same data simultaneously. The first thing that we need to explore is the types of locking in SQL databases.

### Types of Locking in SQL Database

These are two common types of locking in SQL database:

* **Shared Locks** allow multiple transactions to read a resource simultaneously, but prevent other transactions from modifying the locked resource until the lock is released. They are helpful when we need to read data frequently but modify it infrequently.
    
* **Exclusive Locks** are used when a transaction needs to modify data. This type of lock prevents any other transaction from accessing the same data until the lock is released. This means that when a transaction holds an exclusive lock on a resource, it has the ability to modify the data without interference from other transactions.
    

Besides these types of locks, some database also support others such as

* Update locks can be used to protect a resource from being modified while it is being read.
    
* Intent Locks signal the intention to acquire a shared or exclusive lock on a resource. This can be thought of as a lock of locks.
    
* Schema Locks are used to prevent concurrent schema modifications.
    

However, in this document, we are solely focusing on Shared Locks and Exclusive Locks. To continue this topic, we will explore the levels of locking. Unlike types, which focus on controlling behavior, levels are separated by the entity that the lock manages.

### Levels of Locking in SQL Database

Similar to the types, there are several levels of database locking depending on the resource that is locked. These levels of locking can be listed from highest to lowest like following.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682652464211/89147b79-49a5-4975-9cf1-104c3af66f89.png align="center")

Database-level locking limits access to the entire database by other transactions. Therefore, this level of lock is very low and cannot be used for the online version of a multi-user DBMS.

On the other hand, page-level locks (share/exclusive locks) are used to control read/write access to table pages in the shared buffer pool. These locks are released immediately after a row is fetched or updated. Application developers normally need not be concerned with page-level locks.

So this document will focus on database locking at the table and row levels.

**Table level locks** are used to prevent access to a full table or relation by any transactions. The behavior of the lock depends on its type and is not always the same. Generally, these locks are automatically used by the database when proper behavior is triggered. However, you can also acquire a specific lock using the `LOCK` command.

There are several lock modes available for databases, varying in level and type of locking. The key difference between lock types is the set of other lock types that they can conflict with. This means that when a lock is set on a particular table, it prevents other transactions from acquiring conflicting locks. It is important to note that a transaction can conflict with itself.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682652469732/15522638-ffba-4aa8-bee1-62bca948437c.png align="center")

For example in the PostgreSQL, we have following table that represents the Conflicting Locks Modes.

Another common concept is **Row level locks**. At this level, locks do not affect data querying; they only block writers and lockers to the same rows. Row level locks can be released at transaction end or during savepoint rollback, just like table level locks.

Similar to table level locks, row level locks also have different lock modes and each of them may conflict with others. The following table provides a description of these modes:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1682652475487/f52651d4-1828-494e-88a0-3aeade557dfe.png align="center")

This covers the fundamental knowledge about database locking. We will now move on to other interesting parts.

### Advisory locks

In addition to the database-defined locks listed above, some databases provide a means for creating locks that have application-defined meanings, called advisory locks. These locks are not used automatically; sometimes, we need the ability to customize the lock mechanism, so we implement advisory locks on the application level and control them manually.

For example, we can acquire an advisory lock in PostgreSQL in two ways:

* **Advisory lock at the session level**. In this case, the lock is not released automatically after the transaction is done, so we need to release it manually.
    
* **Advisory lock at the transaction level**, which looks more similar to regular locks. We do not need an explicit unlock operator to release it.
    

In the implementation, advisory locks try to acquire an `EXCLUSIVE` lock on a specific relation or table and prevent other transactions from accessing it.

We're good to move on to the next part, where we'll discuss the actual problem.

### Issue with automatic locking usage

As mentioned earlier, table-level locks can be acquired automatically through proper behavior. For example, in PostgreSQL, we can list a few use cases as follows.

* When executing the `SELECT` command, a lock called `ACCESS SHARE` will be placed.
    
* When using row-level locking commands with `SELECT`, such as `SELECT FOR UPDATE`, the `ROW SHARE` lock will be acquired.
    
* The commands `UPDATE`, `DELETE`, `INSERT`, and `MERGE` acquire a `ROW EXCLUSIVE` lock on the target table.
    

Let's return to the original issue of duplicated actions that can occur when multiple applications attempt to modify data in a database concurrently. However, in the previous sentences, we mentioned that an `EXCLUSIVE` lock is acquired when a transaction modifies data. Have you ever wondered why this lock cannot prevent unexpected behaviors that may affect our data?

**Use case Create:** assume 2 customers click to buy the last item at the same time. If status of this goods is available, create invoice then mark the status as sold out. Lock is only acquired on the table invoice, also can’t create unique constraint. And the steps that check status of the goods of both 2 requests are executed in the same time when status is still available.

Let's return to the original issue of duplicated actions that can occur when multiple applications attempt to modify data in a database concurrently. However, in the previous sentences, we mentioned that an `EXCLUSIVE` lock is acquired when a transaction modifies data. Have you ever wondered why this lock cannot prevent unexpected behaviors that may affect our data?

For example, when an user want to buy an item, our system need to do following steps:

* Step 1: Select item from DB where its status is `Available`.
    
* Step 2: Update status to `Sold out` with/without where condition. For example update item status to `Sold out` where item status is `Available` .
    
* Step 3: Create invoice that refer to item ID.
    
* Step 4: Send invoice email to user.
    

Easily to see that at the Step 2, two buy requests go to the server immediately can lead to the conflict when both of them can update successfully if have no addition condition, because when they select item, it is still `Available`.

The first thing that we think about is **Why don’t we use update with condition**. For example: `UPDATE items SET status = 'Sold out' WHERE status = 'Available'`. It is right, basically, this statement can be used to prevent the record from update, an error will be raised to us, then we can handle it. But it is just only can be used in some simple cases when the states of item are clarify.

You can see the first problem that we was raised at the begin is not actually a problem that must need a lock. But it is a simple case that can help us imagine the issue. So, currently, we will go to the more rational case.

Let's consider an example where we have a bank account table that stores the account balance and we need to transfer money from one account to another. To do this, we first need to select the accounts that are involved in the transfer, update their balances, and then commit the transaction.

Using an UPDATE statement with a WHERE clause to update the account balances can lead to data conflicts if multiple transactions try to update the same accounts simultaneously. For example, consider the following scenario:

* Transaction A selects account X with a balance of $1000 and account Y with a balance of $500.
    
* Transaction B also selects account X and account Y at the same time.
    
* Transaction A updates the balance of account X to $800 and the balance of account Y to $700.
    
* Transaction B updates the balance of account X to $900 and the balance of account Y to $600.
    

In this scenario, both transactions have updated the balances of the same accounts at the same time, leading to a data conflict. The final balances of account X and account Y are different depending on which transaction committed first.

To avoid this problem, we can use SELECT FOR UPDATE to lock the rows that we want to update until the transaction is committed. This ensures that only one transaction can modify the selected rows at a time, preventing data conflicts. Here's an example of how we can transfer money using SELECT FOR UPDATE:

```sql
BEGIN TRANSACTION;

SELECT balance FROM accounts WHERE account_number = 'X' FOR UPDATE;
-- Locks the row for account X

SELECT balance FROM accounts WHERE account_number = 'Y' FOR UPDATE;
-- Locks the row for account Y

UPDATE accounts SET balance = balance - 500 WHERE account_number = 'X';
-- Deduct $500 from account X

UPDATE accounts SET balance = balance + 500 WHERE account_number = 'Y';
-- Add $500 to account Y

COMMIT;
-- Releases the locks and commits the transaction
```

In this example, we first use `SELECT FOR UPDATE` to lock the rows for the accounts that we want to update. This ensures that no other transaction can modify these rows until the transaction is committed or rolled back. Then, we update the balances of the accounts and commit the transaction. This ensures that the transaction is atomic and that no other transaction can modify the same accounts at the same time, preventing data conflicts.

How about advisory lock, when we should use this?

Let's say we have a table called `inventory` that contains information about the quantity of items we have in stock. We want to implement a system where multiple transactions can purchase items from the inventory, but we want to ensure that we don't oversell items by allowing transactions to update the same row simultaneously.

One way to prevent overselling is to use `SELECT FOR UPDATE` to lock the relevant rows in the `inventory` table before updating them. However, if two transactions try to update the same row simultaneously, one transaction will block while waiting for the other transaction to release its lock. This can reduce concurrency and potentially cause performance problems if there are many concurrent transactions.

Instead, we can use an advisory lock to ensure that only one transaction can update a row in the `inventory` table at a time. Here's how we can do this:

```sql
BEGIN TRANSACTION;

SELECT pg_advisory_xact_lock(12345);
-- Acquire an advisory lock on a unique identifier (12345 in this example)

SELECT id, quantity FROM inventory WHERE id = 1234 FOR UPDATE;
-- Select the row for the item with ID 1234 and acquire a row-level lock

-- Check that we have enough inventory to fulfill the order
-- ...

-- Update the quantity of the item in the inventory
-- ...

SELECT pg_advisory_unlock(12345);
-- Release the advisory lock

COMMIT;
```

In this example, we first acquire an advisory lock using the `pg_advisory_xact_lock` function. We use a unique identifier (12345 in this example) to ensure that only one transaction can hold the lock at a time.

After acquiring the lock, we select the row for the item with ID 1234 and acquire a row-level lock using the `FOR UPDATE` clause. This ensures that only one transaction can update the row at a time.

We then check that we have enough inventory to fulfill the order and update the quantity of the item in the inventory.

Finally, we release the advisory lock using the `pg_advisory_unlock` function.

Using an advisory lock in this scenario ensures that only one transaction can update a row in the `inventory` table at a time, preventing overselling of items. Because we're only acquiring row-level locks on individual rows when necessary, we can achieve higher concurrency than if we used `SELECT FOR UPDATE` to lock all relevant rows in the `inventory` table.

## Conclusion

In conclusion, concurrency control is an essential aspect of developing applications that can handle multiple user requests simultaneously. SQL databases must implement concurrency control techniques to prevent data inconsistencies and conflicts. In addition to database-defined locks, some databases provide a means for creating locks that have application-defined meanings, called advisory locks, used to customize the lock mechanism for application-specific use-cases. By implementing these concurrency control techniques, software engineers can ensure that their applications can handle a high workload while maintaining performance and uptime.