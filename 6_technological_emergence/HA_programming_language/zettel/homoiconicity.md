# Homoiconicity

## Core Insight
When code and data share the same representation, the boundary between program and data dissolves - the program can examine, modify, and create itself.

Lisp revealed this secret in 1958: what if your code IS your data structure?

```lisp
; This is data: a list
'(+ 1 2)

; This is code: a function call
(+ 1 2)

; The only difference is evaluation
```

In homoiconic languages, programs are written in the language's own data structures. Lisp uses lists. Prolog uses terms. The representation you manipulate as data is exactly the representation that executes as code.

This enables profound metaprogramming:

```lisp
(defmacro when [condition & body]
  `(if ~condition
     (do ~@body)))

; Now we have a new control structure
(when (> x 5)
  (println "x is big")
  (process x))
```

We just extended the language by manipulating code as data. No special parsing. No string manipulation. Just data structure transformation.

Compare with string-based metaprogramming:

```python
code = "def f(x):\n    return x + 1"
exec(code)  # Dangerous, ugly, no structure
```

Versus homoiconic manipulation:

```lisp
(defn make-adder [n]
  `(fn [x] (+ x ~n)))

(def add-five (eval (make-adder 5)))
```

The profound implication: in homoiconic languages, there's no privileged level. Code can inspect code, modify code, generate code, all using the same tools you use for regular data. The language becomes infinitely malleable.

It's the ultimate dissolution of the use-mention distinction. The map becomes the territory.

## Connections
→ [[metaprogramming]]
→ [[code_as_data]]
→ [[macros]]
← [[metalinguistic_abstraction]]

---
Level: L7
Date: 2025-06-22
Tags: #homoiconicity #metaprogramming #lisp #code-as-data