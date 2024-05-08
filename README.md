# goerr
golang error design

I have read [go/issues/21161](https://github.com/golang/go/issues/21161)
My imaging is something similar to that.

before
```go
n, err := io.Write(x)
```

now 
```python
n := io.Write(x) #err
```
