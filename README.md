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
