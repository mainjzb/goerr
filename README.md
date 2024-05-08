# goerr
golang error design

I have read [go/issues/21161](https://github.com/golang/go/issues/21161)
My imaging is something similar to that.

before
```go
n, err := io.Write(x) // 1. err as value
n, _ := io.Write(x)  // 2. ignore error

// 3. return error immediately、
n, err := io.Write(x)
if err != nil {
   return 0, err
}

// 4. wrap additional information
n, err := io.Write(x)
if err != nil {
   return 0, fmt.Error("tcp closed: %w", err)
}
```

now 
```python
n := io.Write(x) #err       // 1. err as value
n := io.Write(x) #@ignore   // 2. ignore error

n := io.Write(x) #@up     //  3. return error immediately、

// 4. wrap additional information
n := io.Write(x) #@wrap(""tcp closed: %w")
```

