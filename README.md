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

// 5. panic err
n, err := io.Write(x)
if err != nil{
    panic(err)
}
```

now 
```python
n := io.Write(x) #err       // 1. err as value

n := io.Write(x) #@ignore   // 2. ignore error

n := io.Write(x) #@done     //  3. return error immediately、

n := io.Write(x) #@wrap("tcp closed: %w") // 4. wrap additional information

n := io.Write(x) #@must     // 5. panic err
```
1. err as value. everything is like before. just error at suffix.
2. ignore error. make it no easier to ignore error than before. Encourage people with additional information.
3. return error immediately、Many times, Especially in libiary. we just need returen error nothing need to do. For example: [url.parseAuthority](https://github.com/golang/go/blob/master/src/net/url/url.go#L586)

5. Many third-party libraries have `MustXxx()` and `Xxx()` two sets api. Now we just need only one.

