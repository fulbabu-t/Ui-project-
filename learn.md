# Learning Notes: UPI Offline Mesh

## Overview

UPI Offline Mesh is a Spring Boot backend that demonstrates how digital payments can be routed through a Bluetooth-style mesh network when internet connectivity is unavailable.

Imagine two users in a basement with no mobile data or Wi-Fi. A sender initiates a payment, which is encrypted and propagated through nearby devices. The payment packet hops from phone to phone until one device eventually gains internet access and uploads the transaction to the backend for settlement.

This project simulates that entire workflow without requiring real Bluetooth hardware.

---

## Key Concepts Learned

### 1. Mesh Networking

A mesh network allows devices to communicate indirectly through intermediate nodes.

Instead of:

```text
Sender → Internet → Server
```

the payment travels as:

```text
Sender → Device A → Device B → Device C → Backend
```

Each device acts as a relay and forwards encrypted packets until they reach a bridge node with internet access.

---

### 2. Hybrid Encryption

The system protects payment data using a combination of:

* RSA-OAEP (public-key encryption)
* AES-256-GCM (symmetric encryption)

Process:

1. Generate a random AES key.
2. Encrypt payment data using AES-GCM.
3. Encrypt the AES key using the server's RSA public key.
4. Send both together as a secure packet.

Benefits:

* Fast encryption
* Strong security
* Tamper detection through GCM authentication tags

---

### 3. Idempotency

A payment packet may reach the backend multiple times through different bridge devices.

Without protection:

```text
Same payment uploaded 3 times
→ Customer charged 3 times
```

Solution:

* Generate SHA-256 hash of the encrypted packet.
* Store hash in an idempotency cache.
* Process only the first occurrence.
* Reject all duplicates.

This guarantees:

```text
One payment
= One settlement
```

---

### 4. Replay Protection

Attackers should not be able to reuse old payment packets.

Protection methods:

* Timestamp validation (`signedAt`)
* Unique nonce for every payment

The backend rejects:

* Expired packets
* Previously processed packets

---

### 5. Transaction Settlement

After validation:

1. Debit sender account
2. Credit receiver account
3. Write transaction to ledger
4. Commit transaction atomically

This ensures data consistency and prevents partial updates.

---

## Backend Technologies

* Java 17
* Spring Boot 3
* Spring Data JPA
* H2 Database
* RSA-OAEP
* AES-256-GCM
* Maven
* JUnit

---

## System Workflow

```text
Sender
   │
   ▼
Create Payment
   │
Encrypt Payload
   │
Create Mesh Packet
   │
   ▼
Bluetooth Mesh Network
   │
   ▼
Bridge Device
   │
Upload Packet
   │
   ▼
Backend
   │
Hash Packet
   │
Deduplicate
   │
Decrypt
   │
Validate
   │
Settle Transaction
   │
   ▼
Ledger Updated
```

---

## What This Project Demonstrates

* Secure offline payment routing
* Hybrid cryptography implementation
* Idempotent transaction processing
* Replay attack prevention
* Distributed system design concepts
* Mesh networking simulation
* Transaction settlement workflows

---

## Main Takeaway

This project demonstrates how encrypted payment instructions can travel through untrusted intermediary devices and still be processed securely, exactly once, when connectivity becomes available. It combines networking, cryptography, concurrency control, and transactional consistency into a single end-to-end system.
