
# Fund/Money Transfer System Database Design

This repository documents a database schema for transferring or sending money between accounts, similar to mobile financial services (MFS) platforms like bKash/paytm.

---

## **Entities and Their Descriptions**

1. **Accounts**  
   - Holds information about user accounts and balances.

2. **Transactions**  
   - Records all money transfer activities (sending, receiving, charges, etc.).

3. **Users**  
   - Stores user information (e.g., phone number, name, etc.).

4. **Transaction Types**  
   - Defines the types of transactions (e.g., send, receive, withdraw, deposit).

5. **Transaction Status**  
   - Defines the possible statuses for transactions (e.g., pending, successful, failed).

---

## **Database Tables**

### 1. **Users**
| Field Name         | Data Type      | Constraints         |
|---------------------|----------------|---------------------|
| `UserID`           | INT (PK)       | Auto Increment      |
| `FullName`         | VARCHAR(100)   | NOT NULL            |
| `PhoneNumber`      | VARCHAR(15)    | UNIQUE, NOT NULL    |
| `Email`            | VARCHAR(100)   | NULLABLE            |
| `PasswordHash`     | VARCHAR(255)   | NOT NULL            |
| `CreatedAt`        | TIMESTAMP      | DEFAULT CURRENT_TIMESTAMP |

---

### 2. **Accounts**
| Field Name         | Data Type      | Constraints         |
|---------------------|----------------|---------------------|
| `AccountID`        | INT (PK)       | Auto Increment      |
| `UserID`           | INT (FK)       | REFERENCES `Users(UserID)` |
| `Balance`          | DECIMAL(18,2)  | DEFAULT 0.00        |
| `Currency`         | VARCHAR(3)     | DEFAULT 'BDT'       |
| `CreatedAt`        | TIMESTAMP      | DEFAULT CURRENT_TIMESTAMP |

---

### 3. **TransactionTypes**
| Field Name         | Data Type      | Constraints         |
|---------------------|----------------|---------------------|
| `TransactionTypeID`| INT (PK)       | Auto Increment      |
| `TypeName`         | VARCHAR(50)    | UNIQUE, NOT NULL    |
| `Description`      | TEXT           | NULLABLE            |

**Sample Data**  
| TransactionTypeID | TypeName     | Description             |
|--------------------|--------------|-------------------------|
| 1                 | Send         | Sending money           |
| 2                 | Receive      | Receiving money         |
| 3                 | Withdraw     | Withdrawing money       |
| 4                 | Deposit      | Depositing money        |

---

### 4. **TransactionStatus**
| Field Name         | Data Type      | Constraints         |
|---------------------|----------------|---------------------|
| `StatusID`         | INT (PK)       | Auto Increment      |
| `StatusName`       | VARCHAR(50)    | UNIQUE, NOT NULL    |
| `Description`      | TEXT           | NULLABLE            |

**Sample Data**  
| StatusID | StatusName  | Description        |
|----------|-------------|--------------------|
| 1        | Pending     | Awaiting process   |
| 2        | Successful  | Completed          |
| 3        | Failed      | Transaction failed |

---

### 5. **Transactions**
| Field Name         | Data Type      | Constraints         |
|---------------------|----------------|---------------------|
| `TransactionID`    | BIGINT (PK)    | Auto Increment      |
| `SenderAccountID`  | INT (FK)       | REFERENCES `Accounts(AccountID)` |
| `ReceiverAccountID`| INT (FK)       | REFERENCES `Accounts(AccountID)` |
| `TransactionTypeID`| INT (FK)       | REFERENCES `TransactionTypes(TransactionTypeID)` |
| `Amount`           | DECIMAL(18,2)  | NOT NULL            |
| `Fee`              | DECIMAL(18,2)  | DEFAULT 0.00        |
| `StatusID`         | INT (FK)       | REFERENCES `TransactionStatus(StatusID)` |
| `CreatedAt`        | TIMESTAMP      | DEFAULT CURRENT_TIMESTAMP |
| `UpdatedAt`        | TIMESTAMP      | ON UPDATE CURRENT_TIMESTAMP |

---

## **Flow Example**
1. **Sending Money**  
   - Deduct the `Amount + Fee` from the `SenderAccountID`.
   - Add the `Amount` to the `ReceiverAccountID`.
   - Record the transaction with type `Send` and `Receive` in the `Transactions` table.
   
2. **Transactional Integrity**  
   Use database transactions to ensure:
   - Sender's balance decreases.
   - Receiver's balance increases.
   - The transaction is recorded atomically.

3. **Transaction History**  
   Query the `Transactions` table to generate user-specific histories (e.g., sent, received, charges).

---

## **Example Queries**

### 1. **Send Money Transaction**
```sql
BEGIN;

-- Deduct from sender
UPDATE Accounts 
SET Balance = Balance - 105.00 -- Amount + Fee
WHERE AccountID = 1;

-- Add to receiver
UPDATE Accounts 
SET Balance = Balance + 100.00 -- Amount
WHERE AccountID = 2;

-- Record transaction
INSERT INTO Transactions 
(SenderAccountID, ReceiverAccountID, TransactionTypeID, Amount, Fee, StatusID) 
VALUES 
(1, 2, 1, 100.00, 5.00, 2); -- Successful

COMMIT;
```

### 2. **Check Account Balance**
```sql
SELECT Balance FROM Accounts WHERE AccountID = 1;
```

### 3. **Transaction History for a User**
```sql
SELECT T.*, TT.TypeName, TS.StatusName 
FROM Transactions T
JOIN TransactionTypes TT ON T.TransactionTypeID = TT.TransactionTypeID
JOIN TransactionStatus TS ON T.StatusID = TS.StatusID
WHERE T.SenderAccountID = 1 OR T.ReceiverAccountID = 1
ORDER BY T.CreatedAt DESC;
```

---

This schema provides a robust **foundation** for building an MFS-like system with scalability, security, and maintainability in mind.
