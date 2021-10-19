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
