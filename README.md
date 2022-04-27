# Invidian Go language style guide

Initially just a list of rules to follow while writing Go code, which are not covered
by already existing linters etc.

## The happy path is left-aligned

https://medium.com/@matryer/line-of-sight-in-code-186dd7cdea88

Bad:

```diff
-  if err == nil {
-    return foo, nil
-  }
-
-  return nil, fmt.Errorf("doing foo: %w", err)
-}
```

Good:

```diff
+  if err != nil {
+    return nil, fmt.Errorf("doing foo: %w", err)
+  }
+
+  return foo, nil
+}
```

## Chained interfaces are not replaceable

Bad:

```diff
- type Foo interface {
-   Print() string
- 	Say() string
-
-   // Don't do this.
-   Bar() Bar
- }

```

Good:

```diff
+ type Foo interface {
+   Print() string
+   Say() string
+ }
+
+ type Printer interface {
+   Print() string
+ }
+
+ func NewBar(f Printer) Bar {
+   return &bar{}
+ }
```

## Mixing exported fields and methods in struct makes it impossible to put it behind an interface on client side

Bad:

```diff
-type Foo struct {
-  ID string
-}
-
-func (f *Foo) Do() error {
-  return nil
-}
```

Good:

```diff
+type Foo struct {
+  id string
+
+func (f *Foo) Do() error {
+  return nil
+}
+
+func (f *Foo) ID() string {
+  return f.id
+}
```

Note: If this problem affects library struct you use, it can be workaround using the following method:

```go
type Foo struct {
  ID string
}

func (f *Foo) Do() error {
  return nil
}

type FooWrapper struct {
	*Foo
}

func (fw *FooWrapper) GetID() string {
	return fw.ID
}
```

## Struct fields are validated if user may put bad values into them or not initialize them

This is a defensive programming approach to avoid misuse or potential panics of programs.

Bad:

```diff
-type Doer interface {
-  Do()
-}
-
-type Foo struct {
-  Doer Doer
-}
-
-func (f *foo) Execute() {
-  f.Doer.Do()
-}
```

Good:

```diff
+type Doer interface {
+  Do()
+}
+
+type Foo interface {
+  Execute()
+}
+
+type foo struct {
+  doer Doer
+}
+
+func NewFoo(doer Doer) (Foo, error) {
+  if doer == nil {
+    return nil, fmt.Errorf("doer can't be nil")
+  }
+
+  return &foo{
+    doer: doer,
+  }, nil
+}
+
+func (f *foo) Execute() {
+  f.doer.Do()
+}
```

## Short if notation should be preferred

It reduces lines of code and scope of variables.

Bad:

```diff
-err := Foo()
-if err != nil {
-  ...
-}
```

Good:

```diff
+if err := Foo(); err != nil {
+  ...
+}
```

## Define exported objects before unexported ones

As a reader, you are more likely to search for exported structs and functions in the code,
as they are higher level as the unexported functions. To make it convinient for the reader,
exported structs, functions etc should be placed at the top of the file.

Bad:

```diff
-func foo() {}
-
-func Bar() {}
```

Good:

```diff
+func Bar() {}
+
+func foo() {}
```

## Don't use 'else'

Most of the time 'else' in 'if' statement can be avoided by either shifting the logic or splitting
the code into functions. This reduces the cognitive complexity while reading code, as instead of
having a path X when Y happens and path Z when Y is negated, you only have path X which executes always
and path Z when Y happens.

More on that: https://medium.com/@jamousgeorges/dont-use-else-66ee163deac3

Bad:

```diff
- var bar int
-
- if foo == 5 {
-   bar = 2
- } else {
-   bar = 3
- }
```

Good:

```diff
+ bar := 3
+
+ if foo != 5 {
+   bar = 2
+ }
```
