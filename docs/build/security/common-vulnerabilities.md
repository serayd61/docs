# Common Vulnerabilities in Clarity

This guide covers common security vulnerabilities in Clarity smart contracts and how to prevent them.

## 1. Access Control Issues

### Vulnerability: Missing Authorization Checks

```clarity
;; VULNERABLE: No access control
(define-public (set-admin (new-admin principal))
  (begin
    (var-set admin new-admin)
    (ok true)))
```

### Fix: Implement Proper Authorization

```clarity
;; SECURE: Proper access control
(define-public (set-admin (new-admin principal))
  (begin
    (asserts! (is-eq tx-sender (var-get admin)) ERR-NOT-AUTHORIZED)
    (var-set admin new-admin)
    (ok true)))
```

## 2. Integer Overflow/Underflow

While Clarity has built-in overflow protection that causes transactions to fail, you should still validate inputs to provide better error messages.

### Vulnerability: Unchecked Arithmetic

```clarity
;; VULNERABLE: Could cause unexpected failures
(define-public (subtract-balance (amount uint))
  (let ((balance (var-get user-balance)))
    (var-set user-balance (- balance amount))
    (ok true)))
```

### Fix: Validate Before Operations

```clarity
;; SECURE: Explicit validation
(define-public (subtract-balance (amount uint))
  (let ((balance (var-get user-balance)))
    (asserts! (>= balance amount) ERR-INSUFFICIENT-BALANCE)
    (var-set user-balance (- balance amount))
    (ok true)))
```

## 3. Improper Use of `tx-sender` vs `contract-caller`

### Understanding the Difference

- `tx-sender`: The original signer of the transaction
- `contract-caller`: The immediate caller (could be another contract)

### Vulnerability: Wrong Principal Check

```clarity
;; VULNERABLE: Could be exploited via contract calls
(define-public (withdraw-funds)
  (let ((balance (map-get? balances contract-caller)))
    ;; Using contract-caller allows other contracts to withdraw
    (match balance
      amount (stx-transfer? amount (as-contract tx-sender) contract-caller)
      ERR-NO-BALANCE)))
```

### Fix: Use Appropriate Principal

```clarity
;; SECURE: Use tx-sender for user funds
(define-public (withdraw-funds)
  (let ((balance (map-get? balances tx-sender)))
    (match balance
      amount (begin
        (map-delete balances tx-sender)
        (as-contract (stx-transfer? amount tx-sender tx-sender)))
      ERR-NO-BALANCE)))
```

## 4. Front-Running Vulnerabilities

### Vulnerability: Predictable Outcomes

```clarity
;; VULNERABLE: Outcome can be predicted and front-run
(define-public (claim-reward)
  (let ((winner (mod block-height u10)))
    (if (is-eq winner u0)
      (ok (award-prize tx-sender))
      (ok false))))
```

### Fix: Use Commit-Reveal Pattern

```clarity
;; SECURE: Commit-reveal prevents front-running
(define-map commitments principal (buff 32))
(define-map reveals principal uint)

(define-public (commit (hash (buff 32)))
  (begin
    (map-set commitments tx-sender hash)
    (ok true)))

(define-public (reveal (secret uint))
  (let ((commitment (map-get? commitments tx-sender)))
    (match commitment
      hash (begin
        (asserts! (is-eq hash (sha256 secret)) ERR-INVALID-REVEAL)
        (map-set reveals tx-sender secret)
        (ok true))
      ERR-NO-COMMITMENT)))
```

## 5. Denial of Service (DoS)

### Vulnerability: Unbounded Loops

```clarity
;; VULNERABLE: Could run out of gas with large lists
(define-public (process-all (items (list 1000 uint)))
  (ok (map process-item items)))
```

### Fix: Implement Pagination

```clarity
;; SECURE: Process in batches
(define-constant BATCH-SIZE u50)

(define-public (process-batch (items (list 50 uint)))
  (begin
    (asserts! (<= (len items) BATCH-SIZE) ERR-BATCH-TOO-LARGE)
    (ok (map process-item items))))
```

## 6. Price Oracle Manipulation

### Vulnerability: Single Oracle Dependency

```clarity
;; VULNERABLE: Single point of failure
(define-public (get-price)
  (contract-call? .single-oracle get-price))
```

### Fix: Use Multiple Oracles with Validation

```clarity
;; SECURE: Multiple oracles with deviation check
(define-public (get-validated-price)
  (let (
    (price1 (unwrap! (contract-call? .oracle-1 get-price) ERR-ORACLE-FAIL))
    (price2 (unwrap! (contract-call? .oracle-2 get-price) ERR-ORACLE-FAIL))
    (price3 (unwrap! (contract-call? .oracle-3 get-price) ERR-ORACLE-FAIL))
    (median (get-median price1 price2 price3))
  )
    ;; Check for excessive deviation
    (asserts! (< (get-deviation price1 median) MAX-DEVIATION) ERR-PRICE-DEVIATION)
    (asserts! (< (get-deviation price2 median) MAX-DEVIATION) ERR-PRICE-DEVIATION)
    (ok median)))
```

## 7. Flash Loan Attacks

### Vulnerability: Instant State Manipulation

```clarity
;; VULNERABLE: Can be manipulated within single transaction
(define-public (vote-with-balance)
  (let ((balance (ft-get-balance .token tx-sender)))
    (map-set votes tx-sender balance)
    (ok true)))
```

### Fix: Time-Weighted Voting

```clarity
;; SECURE: Use time-weighted balance
(define-public (vote-with-balance)
  (let (
    (current-balance (ft-get-balance .token tx-sender))
    (snapshot-balance (get-snapshot-balance tx-sender))
    (voting-power (min current-balance snapshot-balance))
  )
    (map-set votes tx-sender voting-power)
    (ok true)))
```

## 8. Signature Replay

### Vulnerability: Reusable Signatures

```clarity
;; VULNERABLE: Signature can be replayed
(define-public (execute-with-signature (action uint) (signature (buff 65)))
  (begin
    (asserts! (verify-signature action signature) ERR-INVALID-SIG)
    (execute-action action)))
```

### Fix: Include Nonce

```clarity
;; SECURE: Nonce prevents replay
(define-map nonces principal uint)

(define-public (execute-with-signature (action uint) (nonce uint) (signature (buff 65)))
  (let ((expected-nonce (default-to u0 (map-get? nonces tx-sender))))
    (asserts! (is-eq nonce expected-nonce) ERR-INVALID-NONCE)
    (asserts! (verify-signature action nonce signature) ERR-INVALID-SIG)
    (map-set nonces tx-sender (+ nonce u1))
    (execute-action action)))
```

## Security Testing Checklist

For each vulnerability type, ensure you have:

1. **Unit Tests**: Test both valid and invalid inputs
2. **Fuzzing**: Random input testing
3. **Invariant Tests**: Verify state consistency
4. **Integration Tests**: Test contract interactions

## Resources

- [Clarity Security Patterns](https://github.com/serayd61/clarity-patterns)
- [Smart Contract Security Best Practices](https://consensys.github.io/smart-contract-best-practices/)
- [Stacks Bug Bounty Program](https://immunefi.com/bounty/stacks/)
