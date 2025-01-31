---
layout: post
title:  "Race conditions. Have you ever heard about it?"
subtitle: ""
date: 2024-09-22 00:00:00 -0300
background: '/img/posts/race-condition.webp'
tags: [Programming, Golang]
---

Every developer, at some point, needs to understand and know the implications of race conditions in appication development, especially when interacting with databases on the backend.

Imagine your bank account with a balance of USD 100. Now, you receive a PIX payment of USD 200 and simultaneously make a debit purchase of USD 50. If both operations access the balance at the same time, and the race condition is not handled in the application and database, the balance could result in an incorrect value. The first operation checks the balance of USD 100 and, after credit, results in USD 300, completing the operation. Meanwhile, the second operation, occurring simultaneously, also reads the balance of USD 100 and, after debit, results in a final balance of USD 50 in the account, disregarding the previous credit and causing an inconsistency in the persisted balance in the database.

Race conditions occur when multiple operations access and modify data simultaneously without proper synchronization. The result can be unpredictable, causing errors such as inconsistent data in the database or system failures.

To resolve this problem, there are two common approaches: **optimistic locking** and **pessimistic locking** of race conditions.

### Pessimistic Locking
In pessimistic locking, the application assumes that concurrent operations will cause conflicts, so it blocks the shared resource until one operation finishes using it. This type of control ensures that while one transaction is being processed, no other can access the resource, eliminating the possibility of race conditions.

Example in SQL:
```sql
BEGIN;
SELECT balance FROM account WHERE id = 1 FOR UPDATE;
-- Update the balance based on credit or debit logic
UPDATE account SET balance = balance + 200 WHERE id = 1;
COMMIT;
```
In the above example, the `FOR UPDATE` command ensures that the account's balance is locked for other transactions while the current operation is being executed, preventing another transaction from reading or writing to the same record until the `COMMIT` is performed.

### Optimistic Locking
In optimistic locking, the application assumes that conflicts are rare and allows operations to compete for the resource without explicit locks. When an operation is completed, the application checks if the data was modified during the transaction. If it has been, the operation fails, and the application must try again or abort.

Optimistic locking is more efficient in scenarios where concurrency conflict is rare because transactions can be executed simultaneously without locks. However, it requires a version control mechanism or change verification to detect conflicts.

Example with version control (using a `version` column in the database):
```sql
BEGIN;
SELECT balance, version FROM account WHERE id = 1;
-- Check if the version is the same before updating
UPDATE account SET balance = balance + 200, version = version + 1 WHERE id = 1 AND version = 1;
COMMIT;
```
Here, the `version` column is used to ensure that the transaction will only be successful if the record was not altered by another concurrent operation. If the `version` has changed, the transaction fails and must be retried.

### Example using Go of Optimistic Locking

In Go, we can implement the optimistic control strategy by using a function that attempts to perform the operation and handles the error in case of a conflict:

```go
package main

import (
    "database/sql"
    "fmt"
    "log"

    _ "github.com/lib/pq"
)

func updateSaldo(db *sql.DB, id int, amount int) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }
    defer tx.Rollback()

    var balance int
    var version int
    err = tx.QueryRow("SELECT balance, version FROM accounts WHERE id = $1", id).Scan(&balance, &version)
    if err != nil {
        return err
    }

    // Try to update the balance and version
    res, err := tx.Exec("UPDATE accounts SET balance = $1, version = version + 1 WHERE id = $2 AND version = $3", balance+amount, id, version)
    if err != nil {
        return err
    }

    rowsAffected, err := res.RowsAffected()
    if err != nil {
        return err
    }

    if rowsAffected == 0 {
        return fmt.Errorf("version conflict! try again...")
    }

    return tx.Commit()
}

func main() {
    connStr := "user=postgres dbname=teste sslmode=disable"
    db, err := sql.Open("postgres", connStr)
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    err = updateSaldo(db, 1, 200)
    if err != nil {
        fmt.Println("error updating the balance:", err)
    } else {
        fmt.Println("balance updated sucessfuly.")
    }
}
```
In the example above, we used the optimistic approach. If the record version has been modified by another transaction, the application returns an error, indicating that the operation needs to be retried.

### Conclusion
Race conditions can be critical when developing concurrent systems, and choosing the right strategy to handle them is essential. Pessimistic management works well when there is a high probability of conflicts since it locks access to the resource. On the other hand, optimistic management is more efficient when conflicts are rare, allowing greater parallelism in operations.

When dealing with databases and high-concurrency systems, always consider the best approach to ensure data integrity and consistency.