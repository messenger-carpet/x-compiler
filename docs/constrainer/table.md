# Table API Documentation

## Overview

A scoped symbol table implementation that allows for hierarchical symbol binding with scope management. Similar to `java.util.Dictionary` but with scope mechanisms and Symbol-based keys.

## Data Structures

### Binder Class

Internal helper class that manages symbol bindings with scope chain information.

```java
class Binder
```

#### Fields
- `Object value` - The value associated with the symbol
- `Symbol prevtop` - Reference to the previous symbol in the same scope (linked list)
- `Binder tail` - Reference to the previous binder for the same symbol in outer scopes

#### Constructor
```java
Binder(Object v, Symbol p, Binder t)
```

#### Methods
```java
Object getValue()
Symbol getPrevtop()
Binder getTail()
```

### Table Class

Main symbol table implementation with scope management.

```java
public class Table
```

#### Fields
- `HashMap<Symbol,Binder> symbols` - Hash map storing symbol-to-binder mappings
- `Symbol top` - Reference to the last symbol added to current scope
- `Binder marks` - Stack of scope markers for nested scope management

## Public API

### Constructor
```java
public Table()
```
Creates a new empty symbol table.

### Symbol Access Methods

#### get
```java
public Object get(Symbol key)
```
**Parameters:**
- `key` - The Symbol to look up

**Returns:** The Object associated with the specified symbol in the current scope chain

**Description:** Retrieves the value bound to a symbol, following the scope chain from innermost to outermost scope.

#### put
```java
public void put(Symbol key, Object value)
```
**Parameters:**
- `key` - The Symbol to bind
- `value` - The Object to associate with the symbol

**Description:** Binds a value to a symbol in the current scope. Maintains the linked list of symbols in the current scope and preserves previous bindings for the same symbol in outer scopes.

### Scope Management Methods

#### beginScope
```java
public void beginScope()
```
**Description:** Creates a new nested scope. Saves the current state by pushing a new mark onto the mark stack and resetting the top pointer for the new scope.

#### endScope
```java
public void endScope()
```
**Description:** Closes the current scope and restores the table to its state at the most recent `beginScope()`. Removes all symbol bindings added in the current scope and restores previous bindings for symbols that existed in outer scopes.

### Utility Methods

#### keys
```java
public java.util.Set<Symbol> keys()
```
**Returns:** A Set containing all symbols currently in the table

**Description:** Returns the set of all symbols that have bindings in any scope level.

## Memory Diagram Example
```java
table.put(symbolA, objectX);
table.beginScope();
table.put(symbolA, objectY);
```
```
symbols HashMap:
  symbolA -> Binder(objectY, symbolB, Binder(objectX, null, null))
                 ↑           ↑              ↑
               new value   prev in     previous binding
                          scope       (outer scope)
```
                 

## Implementation Notes

- Each symbol binding creates a `Binder` that maintains two linked lists:
  1. **Scope chain**: Links to the previous symbol added in the same scope
  2. **Symbol chain**: Links to the previous binding of the same symbol in outer scopes

- The `top` field maintains a reference to the most recently added symbol in the current scope

- The `marks` field implements a stack of scope boundaries using the `Binder` structure

- When a scope ends, all symbols added in that scope are removed, and previous bindings for shadowed symbols are restored
