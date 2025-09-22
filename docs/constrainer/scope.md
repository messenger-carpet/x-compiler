# BeginScope and EndScope Operations

This document demonstrates the `beginScope()` and `endScope()` operations in detail, showing how the scope stack mechanism works with multiple nested scopes.

## Code Analysis

### beginScope() Implementation
```java
public void beginScope() {
    marks = new Binder(null, top, marks);
    top = null;
}
```

### endScope() Implementation
```java
public void endScope() {
    while (top != null) {
        Binder e = symbols.get(top);
        if (e.getTail() != null) 
            symbols.put(top, e.getTail());
        else 
            symbols.remove(top);
        top = e.getPrevtop();
    }
    top = marks.getPrevtop();
    marks = marks.getTail();
}
```

---

## Example 1: Single Nested Scope

### Setup: Create outer scope
```java
Table t = new Table();
t.put(Symbol("x"), "outer-x");
t.put(Symbol("y"), "outer-y");
```

**State after setup**:
```
symbols: {
  Symbol("x") -> Binder("outer-x", null, null),
  Symbol("y") -> Binder("outer-y", Symbol("x"), null)
}
top: Symbol("y")
marks: null
```

**Scope chain**: `top -> Symbol("y") -> Symbol("x") -> null`

---

### Operation: `beginScope()`

**Step-by-step execution**:
1. Create new mark: `new Binder(null, Symbol("y"), null)`
2. Update marks: `marks = new_mark`
3. Reset top: `top = null`

**State after beginScope()**:
```
symbols: {
  Symbol("x") -> Binder("outer-x", null, null),
  Symbol("y") -> Binder("outer-y", Symbol("x"), null)
}
top: null
marks: Binder(null, Symbol("y"), null)
```

**Visualization**:
```
Current Scope: top -> null (empty inner scope)
Scope Stack:   marks -> [saved_top=Symbol("y"), previous_mark=null]
Outer Scope:   Symbol("y") -> Symbol("x") -> null (preserved)
```

---

### Add symbols to inner scope
```java
t.put(Symbol("x"), "inner-x");  // shadows outer x
t.put(Symbol("z"), "inner-z");  // new symbol
```

**State after inner puts**:
```
symbols: {
  Symbol("x") -> Binder("inner-x", null, Binder("outer-x", null, null)),
  Symbol("y") -> Binder("outer-y", Symbol("x"), null),
  Symbol("z") -> Binder("inner-z", Symbol("x"), null)
}
top: Symbol("z")
marks: Binder(null, Symbol("y"), null)
```

**Visualization**:
```
Current Scope: top -> Symbol("z") -> Symbol("x") -> null
Symbol Chains:
  Symbol("x"): [inner-x] -> [outer-x] -> null
  Symbol("y"): [outer-y] -> null
  Symbol("z"): [inner-z] -> null
```

---

### Operation: `endScope()`

**Step-by-step execution**:

**While Loop - Iteration 1** (`top = Symbol("z")`):
1. Get binder: `e = symbols.get(Symbol("z"))` = `Binder("inner-z", Symbol("x"), null)`
2. Check tail: `e.getTail()` = `null`
3. Remove symbol: `symbols.remove(Symbol("z"))`
4. Move to next: `top = e.getPrevtop()` = `Symbol("x")`

**While Loop - Iteration 2** (`top = Symbol("x")`):
1. Get binder: `e = symbols.get(Symbol("x"))` = `Binder("inner-x", null, Binder("outer-x", null, null))`
2. Check tail: `e.getTail()` = `Binder("outer-x", null, null)`
3. Restore previous: `symbols.put(Symbol("x"), tail_binder)`
4. Move to next: `top = e.getPrevtop()` = `null`

**While Loop - End** (`top = null`):
- Loop terminates

**Scope Restoration**:
1. Restore top: `top = marks.getPrevtop()` = `Symbol("y")`
2. Restore marks: `marks = marks.getTail()` = `null`

**Final State**:
```
symbols: {
  Symbol("x") -> Binder("outer-x", null, null),
  Symbol("y") -> Binder("outer-y", Symbol("x"), null)
}
top: Symbol("y")
marks: null
```

**Result**: Perfectly restored to pre-beginScope() state! ✓

---

## Example 2: Multiple Nested Scopes

### Setup: Three-level nesting
```java
Table t = new Table();

// Level 1: Global scope
t.put(Symbol("a"), "global-a");
t.put(Symbol("b"), "global-b");

// Level 2: First nested scope
t.beginScope();
t.put(Symbol("a"), "level2-a");
t.put(Symbol("c"), "level2-c");

// Level 3: Second nested scope  
t.beginScope();
t.put(Symbol("a"), "level3-a");
t.put(Symbol("d"), "level3-d");
```

**State after all operations**:
```
symbols: {
  Symbol("a") -> Binder("level3-a", null, Binder("level2-a", null, Binder("global-a", null, null))),
  Symbol("b") -> Binder("global-b", Symbol("a"), null),
  Symbol("c") -> Binder("level2-c", Symbol("a"), null),
  Symbol("d") -> Binder("level3-d", Symbol("a"), null)
}
top: Symbol("d")
marks: Binder(null, Symbol("c"), Binder(null, Symbol("b"), null))
```

**Scope Stack Visualization**:
```
Level 3 (current): top -> Symbol("d") -> Symbol("a") -> null
Level 2 (saved):   marks -> [saved_top=Symbol("c"), previous_mark=->]
Level 1 (saved):   previous_mark -> [saved_top=Symbol("b"), previous_mark=null]

Symbol("a") chain: [level3-a] -> [level2-a] -> [global-a] -> null
```

---

### First endScope(): Close Level 3

**While Loop Processing**:
1. **Symbol("d")**: `tail=null` → `remove(Symbol("d"))`
2. **Symbol("a")**: `tail=Binder("level2-a",...)` → restore level2 binding

**Scope Restoration**:
- `top = marks.getPrevtop()` = `Symbol("c")`
- `marks = marks.getTail()` = `Binder(null, Symbol("b"), null)`

**State after first endScope()**:
```
symbols: {
  Symbol("a") -> Binder("level2-a", null, Binder("global-a", null, null)),
  Symbol("b") -> Binder("global-b", Symbol("a"), null),
  Symbol("c") -> Binder("level2-c", Symbol("a"), null)
}
top: Symbol("c")
marks: Binder(null, Symbol("b"), null)
```

---

### Second endScope(): Close Level 2

**While Loop Processing**:
1. **Symbol("c")**: `tail=null` → `remove(Symbol("c"))`
2. **Symbol("a")**: `tail=Binder("global-a",...)` → restore global binding

**Scope Restoration**:
- `top = marks.getPrevtop()` = `Symbol("b")`
- `marks = marks.getTail()` = `null`

**Final State**:
```
symbols: {
  Symbol("a") -> Binder("global-a", null, null),
  Symbol("b") -> Binder("global-b", Symbol("a"), null)
}
top: Symbol("b")
marks: null
```

**Result**: Back to original global scope! ✓

---


This implementation provides robust lexical scoping semantics essential for programming language implementations!
