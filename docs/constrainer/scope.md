```java
Symbol s = Symbol.symbol("a"),
            s1 = Symbol.symbol("b"),
            s2 = Symbol.symbol("c");

        Table t = new Table();
        t.beginScope();
        t.put(s,1);
        t.put(s1,2);
        t.beginScope();
        t.put(s,3);
        t.put(s1,4");
        t.endScope();
        t.endScope();
```
```
marks: Binder(null, null, null)
top: null
symbols: { s ->  Binder(1, null, null) }
top: s
symbols: { s1 -> Binder(2, s, null) }
top: s1
marks: Binder(null, s1, Binder(null, null, null)
top: null
symbols: { s -> Binder(3, null, Binder(1, null, null) }
top: s
symbols: { s1 -> Binder(4, s, Binder(2, s, null)) }
top: s1
e: Binder(4, s, Binder(2, s, null)
symbols: {s1 -> Binder(2, s, null) }
top: s
e: Binder(3, null, Binder(1, null, null) }
symbols: {s -> Binder(1, null, null) }
top: null
top: s1
marks: Binder(null, null, null)
e: Binder(2, s, null)
symbols: { s }
top: s
e: Binder(1, null, null)
symbols: {}
top: null
top: null
marks: null
