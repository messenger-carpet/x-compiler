
# Complete Memory Diagram: Nested Scopes Example

## Code Sequence
```java
Symbol s = Symbol.symbol("a");
Symbol s1 = Symbol.symbol("b"); 
Symbol s2 = Symbol.symbol("c");
Table t = new Table();
t.beginScope();
t.put(s, 1);
t.put(s1, 2);
t.beginScope();
t.put(s, 3);
t.put(s1, 4);
t.endScope();
t.endScope();
```

## Step-by-Step Memory States

### Initial State
```
Table t = new Table();

marks: null
top: null
symbols: {} (empty HashMap)
```

---

### After `t.beginScope();` (Start Scope 1)
```java
marks = new Binder(null, top, marks);
top = null;
```

```
marks: Binder(null, null, null)
       ↑     ↑     ↑
    value prevtop tail
    (null)(null) (null)

top: null
symbols: {} (empty HashMap)
```

---

### After `t.put(s, 1);`
```java
symbols.put(s, new Binder(1, top, symbols.get(s)));
top = s;
```

```
marks: Binder(null, null, null)
top: s
symbols: { 
  s -> Binder(1, null, null)
       ↑     ↑     ↑
    value prevtop tail
    (1)   (null) (null)
}
```

---

### After `t.put(s1, 2);`
```java
symbols.put(s1, new Binder(2, top, symbols.get(s1)));
top = s1;
```

```
marks: Binder(null, null, null)
top: s1
symbols: { 
  s  -> Binder(1, null, null)
  s1 -> Binder(2, s, null)
        ↑     ↑  ↑
     value prevtop tail
     (2)   (s)   (null)
}

Scope 1 chain: s1 -> s -> null
```

---

### After `t.beginScope();` (Start Scope 2)
```java
marks = new Binder(null, top, marks);
top = null;
```

```
marks: Binder(null, s1, Binder(null, null, null))
       ↑     ↑   ↑
    value prevtop tail
    (null)(s1)  (previous mark)

top: null
symbols: { 
  s  -> Binder(1, null, null)
  s1 -> Binder(2, s, null)
}
```

---

### After `t.put(s, 3);` (Shadow s)
```java
symbols.put(s, new Binder(3, top, symbols.get(s)));
top = s;
```

```
marks: Binder(null, s1, Binder(null, null, null))
top: s
symbols: { 
  s  -> Binder(3, null, Binder(1, null, null))
        ↑     ↑     ↑
     value prevtop tail (PREVIOUS BINDING from scope 1!)
     (3)   (null) 
     
  s1 -> Binder(2, s, null)
}
```

---

### After `t.put(s1, 4);` (Shadow s1)
```java
symbols.put(s1, new Binder(4, top, symbols.get(s1)));
top = s1;
```

```
marks: Binder(null, s1, Binder(null, null, null))
top: s1
symbols: { 
  s  -> Binder(3, null, Binder(1, null, null))
  
  s1 -> Binder(4, s, Binder(2, s, null))
        ↑     ↑  ↑
     value prevtop tail (PREVIOUS BINDING from scope 1!)
     (4)   (s)
}

Scope 2 chain: s1 -> s -> null
```

---

## First `t.endScope();` (End Scope 2)

### Processing s1 (top symbol)
```java
while (top != null) {
    Binder e = symbols.get(top);  // e = Binder(4, s, Binder(2, s, null))
    if (e.getTail() != null) symbols.put(top, e.getTail());
    else symbols.remove(top);
    top = e.getPrevtop();
}
```

**Step 1**: Process s1
```
e: Binder(4, s, Binder(2, s, null))
   ↑     ↑  ↑
getValue getPrevtop getTail

tail is not null, so:
symbols.put(s1, Binder(2, s, null))  // Restore scope 1 binding
top = s  // Move to previous symbol in scope 2
```

```
marks: Binder(null, s1, Binder(null, null, null))
top: s
symbols: { 
  s  -> Binder(3, null, Binder(1, null, null))
  s1 -> Binder(2, s, null)  ← RESTORED to scope 1 value
}
```

---

### Processing s (remaining symbol in scope 2)
**Step 2**: Process s
```
e: Binder(3, null, Binder(1, null, null))

tail is not null, so:
symbols.put(s, Binder(1, null, null))  // Restore scope 1 binding
top = null  // Move to previous symbol (end of scope 2 chain)
```

```
marks: Binder(null, s1, Binder(null, null, null))
top: null
symbols: { 
  s  -> Binder(1, null, null)  ← RESTORED to scope 1 value
  s1 -> Binder(2, s, null)
}
```

---

### Restore scope 1 state
```java
top = marks.getPrevtop();  // top = s1
marks = marks.getTail();   // marks = Binder(null, null, null)
```

```
marks: Binder(null, null, null)
top: s1
symbols: { 
  s  -> Binder(1, null, null)
  s1 -> Binder(2, s, null)
}

// Back to exact state after scope 1 was populated!
```

---

## Second `t.endScope();` (End Scope 1)

### Processing s1 (top symbol in scope 1)
**Step 1**: Process s1
```
e: Binder(2, s, null)

tail is null, so:
symbols.remove(s1)  // Remove s1 completely
top = s  // Move to previous symbol in scope 1
```

```
marks: Binder(null, null, null)
top: s
symbols: { 
  s -> Binder(1, null, null)
}
```

---

### Processing s (remaining symbol in scope 1)
**Step 2**: Process s
```
e: Binder(1, null, null)

tail is null, so:
symbols.remove(s)  // Remove s completely
top = null  // Move to previous symbol (end of scope 1 chain)
```

```
marks: Binder(null, null, null)
top: null
symbols: {} (empty HashMap)
```

---

### Restore global state
```java
top = marks.getPrevtop();  // top = null
marks = marks.getTail();   // marks = null
```

## Final State
```
marks: null
top: null
symbols: {} (empty HashMap)

// Back to initial state - all scopes properly cleaned up!
```
