# **Invariant Thinking (Smart Contract Security)**

This module focuses on building a formal mental model of contract behavior using **invariants, pre-conditions, post-conditions, and state transitions**.
These concepts form the foundation of professional auditing and formal verification.

---

## **📌 1. Global Invariants (By Module)**

---

### ### **ERC20 — Safety & State Invariants**

* `sum(balances) == totalSupply`
* `balance[addr] >= 0`
* `balance[addr] <= totalSupply`
* `allowance[owner][spender] >= 0`
* `totalSupply >= 0`
* `owner != address(0)` *(when Ownable included)*
* `totalSupply <= type(uint256).max`

---

### **Vault — Accounting Invariants**

* `totalDeposits >= 0`
* `balance[user] >= 0` for all users
* `sum(balances) == totalDeposits`
* `address(this).balance == totalDeposits`
* `owner != address(0)`
* No unauthorized minting of balances

---

### **Ownable — Authorization Invariants**

* `owner != address(0)`
* Only `owner` can call `onlyOwner` functions
* Ownership changes only via `transferOwnership()`
* `newOwner != address(0)`
* Old owner loses access immediately after transfer

---

### **Pausable — Execution Invariants**

* `paused ∈ {true, false}`
* If `paused == true`: all `whenNotPaused` functions revert
* If `paused == false`: all `whenPaused` functions revert
* Pause state changes only via `pause()` / `unpause()`
* Only authorized roles can toggle pause state

---

---

## **📌 2. Pre-Conditions & Post-Conditions**

---

### **deposit()**

**Pre-conditions**

* `msg.value > 0`
* `msg.sender != address(0)`

**Post-conditions**

* `balance[msg.sender] == oldBalance + msg.value`
* `totalDeposits == oldTotal + msg.value`
* `sum(balances) == totalDeposits`
* No other balances modified

---

### **withdraw()**

**Pre-conditions**

* `amount > 0`
* `balance[msg.sender] >= amount`

**Post-conditions**

* `balance[msg.sender] == oldBalance - amount`
* `totalDeposits == oldTotal - amount`
* `sum(balances) == totalDeposits`

---

### **transfer() (ERC20)**

**Pre-conditions**

* `from != to`
* `to != address(0)`
* `amount <= balance[from]`

**Post-conditions**

* `balance[from] == oldFromBalance - amount`
* `balance[to] == oldToBalance + amount`
* `totalSupply` unchanged

---

### **approve()**

**Pre-conditions**

* `spender != address(0)`

**Post-conditions**

* `allowance[msg.sender][spender] == newValue`
* No balances modified

---

### **transferFrom()**

**Pre-conditions**

* `amount <= balance[from]`
* `allowance[from][msg.sender] >= amount`
* `to != address(0)`
* `from != to`

**Post-conditions**

* `balance[from] == oldFromBalance - amount`
* `balance[to] == oldToBalance + amount`
* `allowance[from][msg.sender] == oldAllowance - amount`
* `totalSupply` unchanged

---

### **emergencyWithdraw() (Vault)**

**Pre-conditions**

* Caller is `owner`
* `amount <= address(this).balance`

**Post-conditions**

* Ether goes only to owner
* `totalDeposits == 0`
* User balances unchanged

---

---

## **📌 3. State Transition Diagrams**

---

### **Vault**

```
Idle
  │
  ├── deposit() ───► Depositing ───► Idle
  │
  └── withdraw() ─► Withdrawing ─► Idle
```

---

### **Pausable**

```
Unpaused ── pause() ──► Paused ── unpause() ──► Unpaused
```

---

### **Ownable**

```
owner = Alice
      │
      └── transferOwnership(Bob)
      │
owner = Bob
```

---

---

## **📌 4. Formal Invariant Annotations (Scribble / Certora Style)**

### **Vault**

```solidity
/// #invariant sum(balances) == totalDeposits
/// #invariant address(this).balance == totalDeposits
/// #invariant owner != address(0)
```

### **ERC20**

```solidity
/// #invariant sum(balances) == totalSupply
/// #invariant totalSupply >= 0
/// #invariant allowance[msg.sender][spender] >= 0
```

### **Pausable**

```solidity
/// #invariant paused == true || paused == false
```

### **Ownable**

```solidity
/// #invariant owner != address(0)
```

---

## **📌 5. Purpose of Week 2**

By the end of this module, you should be able to:

* Think in invariants like a professional auditor
* Predict failures before they occur
* Evaluate whether state transitions are safe
* Write 5–7 invariants for any smart contract
* Document all pre/post-conditions for correctness
* Build a mental model resilient to attacker behavior



