# Abstract Syntax Tree Examples

## Overview

This recursive-descent parser builds Abstract Syntax Trees (ASTs) for a simple programming language. The parser respects operator precedence and associativity rules as defined in the grammar.

## Expression Parsing Rules

The grammar defines three levels of expressions with increasing precedence:

1. **Expression (E)** - Relational operators: `==`, `!=`, `<`, `<=`
2. **Simple Expression (SE)** - Adding operators: `+`, `-`, `|` (or)
3. **Term (T)** - Multiplying operators: `*`, `/`, `&` (and)
4. **Factor (F)** - Primary expressions: parentheses, names, integers, function calls

## Key Precedence Rules

- **Multiplication/Division** have higher precedence than **Addition/Subtraction**
- **Left-associative**: operators of the same precedence are evaluated left-to-right
- **Parentheses** override precedence rules

## AST Examples

### Example 1: `A + B * C`

**Input Expression:** `A + B * C`

**Parsing Process:**
1. `rSimpleExpr()` calls `rTerm()` for first operand
2. `rTerm()` parses `A` as a factor
3. Back in `rSimpleExpr()`, finds `+` operator
4. `rTerm()` is called again for right operand
5. `rTerm()` parses `B * C` as a complete term (multiplication has higher precedence)

**Resulting AST:**
```
      +
     / \
    A   *
       / \
      B   C
```

**Evaluation Order:** `B * C` first, then `A + (result)`

### Example 2: `A * B + C`

**Input Expression:** `A * B + C`

**Parsing Process:**
1. `rSimpleExpr()` calls `rTerm()` for first operand
2. `rTerm()` parses `A * B` as a complete term (left-associative)
3. Back in `rSimpleExpr()`, finds `+` operator
4. `rTerm()` is called for right operand `C`

**Resulting AST:**
```
      +
     / \
    *   C
   / \
  A   B
```

**Evaluation Order:** `A * B` first, then `(result) + C`

### Example 3: `A + B + C` (Left Associativity)

**Input Expression:** `A + B + C`

**Parsing Process:**
1. First `+` creates a tree with `A` and `B`
2. Second `+` creates a new tree with the previous result and `C`

**Resulting AST:**
```
        +
       / \
      +   C
     / \
    A   B
```

**Evaluation Order:** `A + B` first, then `(result) + C`

### Example 4: `A * B * C` (Left Associativity)

**Input Expression:** `A * B * C`

**Resulting AST:**
```
        *
       / \
      *   C
     / \
    A   B
```

### Example 5: `(A + B) * C` (Parentheses Override Precedence)

**Input Expression:** `(A + B) * C`

**Resulting AST:**
```
      *
     / \
    +   C
   / \
  A   B
```

**Evaluation Order:** `A + B` first (due to parentheses), then `(result) * C`

## Parser Methods Responsible for Precedence

### `rSimpleExpr()` - Handles Addition/Subtraction
```java
public AST rSimpleExpr() throws SyntaxError {
    AST t, kid = rTerm();  // Get first term
    while ( (t = getAddOperTree()) != null) {  // While more +/- operators
        t.addKid(kid);         // Previous result becomes left child
        t.addKid(rTerm());     // New term becomes right child
        kid = t;               // New tree becomes current result
    }
    return kid;
}
```

### `rTerm()` - Handles Multiplication/Division
```java
public AST rTerm() throws SyntaxError {
    AST t, kid = rFactor();  // Get first factor
    while ( (t = getMultOperTree()) != null) {  // While more */& operators
        t.addKid(kid);           // Previous result becomes left child
        t.addKid(rFactor());     // New factor becomes right child
        kid = t;                 // New tree becomes current result
    }
    return kid;
}
```

## Parentheses Evaluation with Call Stack Visualization

### Example: `(A + B) * (C + D)`

**Input Expression:** `(A + B) * (C + D)`

Let's trace through the parser's call stack to see how parentheses are handled:

#### Resulting AST:
```
        *
       / \
      +   +
     / \ / \
    A  B C  D
```

### Key Insights About Parentheses Handling:

1. **Recursive Calls:** Each `(` triggers a recursive call to `rExpr()`, creating a new stack frame
2. **Priority Override:** The recursive call ensures expressions in parentheses are fully evaluated before returning to the outer context
3. **Natural Precedence:** The call stack naturally implements precedence - inner expressions must complete before outer ones
4. **Stack Unwinding:** As the stack unwinds, it builds the AST from the inside out

### Another Example: `A + (B * C + D) * E`

**Input Expression:** `A + (B * C + D) * E`

#### Parsing Steps:
1. Parse `A` (simple factor)
2. Find `+` operator
3. Parse right side: `(B * C + D) * E`
4. **Recursive call** for `(B * C + D)`:
   - Parse `B * C` (multiplication first)
   - Parse `+ D` (addition)
   - Return complete subtree
5. Continue with `* E`

#### Resulting AST:
```
      +
     / \
    A   *
       / \
      +   E
     / \
    *   D
   / \
  B   C
```

## Complex Expression Example: `A + B * C + D * E`

**Input Expression:** `A + B * C + D * E`

**Resulting AST:**
```
          +
         / \
        +   *
       / \  / \
      A  * D  E
        / \
       B   C
```

**Evaluation Order:** 
1. `B * C`
2. `A + (B * C)`
3. `D * E`
4. `(A + B * C) + (D * E)`

## Grammar Rules Summary

The precedence hierarchy (highest to lowest):
1. **Parentheses** `( )`
2. **Multiplication, Division, And** `*`, `/`, `&`
3. **Addition, Subtraction, Or** `+`, `-`, `|`
4. **Relational** `==`, `!=`, `<`, `<=`

All operators at the same level are **left-associative**, meaning they evaluate from left to right when precedence is equal.
