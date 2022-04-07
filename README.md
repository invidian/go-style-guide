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
