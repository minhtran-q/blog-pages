---
title: "Optimistic Lock with @Transactional’s Spring Data JPA in Cockroach Database"
date: 2023-11-21
---

_Attention: All commands and examples below are valid for CockroachDb only. Because CockroachDb only support the highest isolation level (Serializable)._

### Introduction

Optimistic locking is a concurrency control strategy used in databases and applications to ensure data integrity. And what if we combine it with `@Transactional` in Spring Data JPA? 
In this blog post, let's take a deeper look at how them work.

### Optimistic Locking
Optimistic locking, also known as Optimistic concurrency control (OCC) is a concurrency control method applied to transactional systems.
Before committing, each transaction verifies that no other transaction has modified the data it has read. If the check reveals conflicting modifications, the committing transaction rolls back and can be restarted.

**In JPA:**

In order to use optimistic locking, we need to have an entity including a property with `@Version` annotation.
If the value has changed in the meantime, an `OptimisticLockException` is thrown. Otherwise, the transaction commits the update and increments a value version property.

For example:

```
@Entity
public class Student {

    @Id
    private Long id;
    private String name;
    private String lastName;
    @Version
    private Integer version;

    // getters and setters
}
```

### @Transactional annotation

Each time you mark the @Transactional annotation on a method.

```
@Transactional
public void registerAccount() {
   // business code
}
```
You should remember that when you call such a method, the invocation in fact will be wrapped with a transaction handling code similar to this:
```
UserTransaction userTransaction = entityManager.getTransaction();
try {
  // begin a new transaction
   userTransaction.begin(); 

   registerAccount(); // the actual method invocation

   userTransaction.commit();
} catch(RuntimeException e) {
   userTransaction.rollback(); // initiate rollback if business code fails
   throw e;
}
```
Each method with @Transactional will be executed in a separate transaction. It also means that each SQL statement will be executed in a separate transaction.

In case you have to update multiple tables or avoid LazyLoadException. You will have to mark a @Transactional outside a wrapper method. Now you will have all SQL statements executed in a single transaction.

_Example:_
```
@Transactional
public void changeStatus(long id, String status) {
  Account account = accountRepository.findById(id);
  account.setStatus(status);
  accountRepository.save(account);
}
```
In this example, we have two queries that will be executed against the database. And since we have marked `@Transactional` on this method, those queries will be executed in _one_ transaction.

Now we have two ways to use @Transactional annotation. And how they work with Optismick Lock mechanism?

### Optimistic Lock and @Transactional annotation

To answer the question above, let's look at the example below.

