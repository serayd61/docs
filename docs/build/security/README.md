# Security Best Practices

Building secure smart contracts on Stacks requires understanding common vulnerabilities and implementing proper safeguards. This guide covers essential security patterns for Clarity developers.

## Overview

Clarity's design provides inherent security advantages:
- **No reentrancy by default**: Clarity's execution model prevents traditional reentrancy attacks
- **Decidable language**: All Clarity programs terminate, preventing infinite loops
- **No null references**: Clarity uses explicit optionals instead of null values
- **Arithmetic safety**: Built-in overflow/underflow protection

However, developers must still implement proper security measures for:
- Access control
- Input validation
- State management
- Economic security

## Quick Links

| Topic | Description |
|-------|-------------|
| [Access Control](./access-control.md) | Role-based permissions and ownership patterns |
| [Input Validation](./input-validation.md) | Validating user inputs and parameters |
| [Common Vulnerabilities](./common-vulnerabilities.md) | Known attack vectors and mitigations |
| [Audit Checklist](./audit-checklist.md) | Pre-deployment security review |

## Key Principles

### 1. Principle of Least Privilege
Only grant the minimum permissions necessary for each function or role.

```clarity
;; Bad: Anyone can call admin functions
(define-public (admin-action)
  (ok (do-sensitive-thing)))

;; Good: Restricted to authorized principals
(define-public (admin-action)
  (begin
    (asserts! (is-admin tx-sender) ERR-NOT-AUTHORIZED)
    (ok (do-sensitive-thing))))
```

### 2. Fail Securely
When errors occur, fail in a way that doesn't leave the contract in an inconsistent state.

```clarity
;; Use asserts! to revert entire transaction on failure
(define-public (transfer (amount uint) (recipient principal))
  (begin
    (asserts! (> amount u0) ERR-INVALID-AMOUNT)
    (asserts! (not (is-eq recipient tx-sender)) ERR-SELF-TRANSFER)
    (try! (stx-transfer? amount tx-sender recipient))
    (ok true)))
```

### 3. Check-Effects-Interactions Pattern
Update state before making external calls to prevent unexpected behavior.

```clarity
(define-public (withdraw (amount uint))
  (let ((current-balance (get-balance tx-sender)))
    ;; 1. CHECKS
    (asserts! (>= current-balance amount) ERR-INSUFFICIENT-BALANCE)
    
    ;; 2. EFFECTS (update state first)
    (map-set balances tx-sender (- current-balance amount))
    
    ;; 3. INTERACTIONS (external call last)
    (as-contract (stx-transfer? amount tx-sender tx-sender))))
```

### 4. Defense in Depth
Implement multiple layers of security rather than relying on a single check.

```clarity
(define-public (critical-operation (data uint))
  (begin
    ;; Layer 1: Access control
    (asserts! (is-authorized tx-sender) ERR-NOT-AUTHORIZED)
    
    ;; Layer 2: Rate limiting
    (asserts! (not (is-rate-limited tx-sender)) ERR-RATE-LIMITED)
    
    ;; Layer 3: Input validation
    (asserts! (is-valid-input data) ERR-INVALID-INPUT)
    
    ;; Layer 4: State validation
    (asserts! (is-valid-state) ERR-INVALID-STATE)
    
    (ok (process-operation data))))
```

## Common Patterns

### Ownership Pattern

```clarity
(define-data-var contract-owner principal tx-sender)

(define-read-only (get-owner)
  (var-get contract-owner))

(define-public (transfer-ownership (new-owner principal))
  (begin
    (asserts! (is-eq tx-sender (var-get contract-owner)) ERR-NOT-OWNER)
    (var-set contract-owner new-owner)
    (ok true)))
```

### Pausable Pattern

```clarity
(define-data-var is-paused bool false)

(define-public (pause)
  (begin
    (asserts! (is-owner) ERR-NOT-AUTHORIZED)
    (var-set is-paused true)
    (ok true)))

(define-public (protected-function)
  (begin
    (asserts! (not (var-get is-paused)) ERR-PAUSED)
    ;; Function logic
    (ok true)))
```

### Timelock Pattern

```clarity
(define-map pending-actions uint {
  action: (string-ascii 64),
  execute-after: uint,
  executed: bool
})

(define-constant TIMELOCK-DELAY u144) ;; ~24 hours

(define-public (queue-action (action-id uint) (action (string-ascii 64)))
  (begin
    (asserts! (is-admin tx-sender) ERR-NOT-AUTHORIZED)
    (map-set pending-actions action-id {
      action: action,
      execute-after: (+ block-height TIMELOCK-DELAY),
      executed: false
    })
    (ok true)))
```

## Security Checklist

Before deploying any contract:

- [ ] All public functions have proper access control
- [ ] Input validation on all user-provided data
- [ ] State changes follow check-effects-interactions pattern
- [ ] No hardcoded secrets or private keys
- [ ] Emergency pause mechanism implemented
- [ ] Upgrade path considered (if needed)
- [ ] Comprehensive test coverage
- [ ] Independent security audit completed

## Resources

- [Clarity Language Reference](https://docs.stacks.co/clarity)
- [SIP-010 Token Standard](https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md)
- [Clarity Security Patterns](https://github.com/serayd61/clarity-patterns)
- [Stacks Security Advisories](https://github.com/stacks-network/stacks-blockchain/security/advisories)
