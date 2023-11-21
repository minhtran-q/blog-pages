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



### @Transactional annotation

Each time you mark the @Transactional annotation on a method. you should remember that when you call such a method, the invocation in fact will be wrapped with a transaction handling code similar to this:

-- code example

Each method with @Transactional will be executed in a separate transaction. It also means that each SQL statement will be executed in a separate transaction.
In case you have to update multiple tables or avoid LazyLoadException. You will have to mark a @Transactional outside a wrapper method. Now you will have all SQL statements executed in a single transaction.

### Optimistic Lock and @Transactional annotation
