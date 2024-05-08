# goerr
golang error design

I have read [go/issues/21161](https://github.com/golang/go/issues/21161)
My imaging is something similar to that.

before
```
n,err := io.Write(x)
```

now 
```
n := io.Write(x) #err
```
