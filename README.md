# `#err` error suffix design
English is not my native language, so it may be a bit strange for you to read this article

Make `#` as error symbol, I believe people can learn and understand it quickly. (Maybe other symbol replace it)

`#@` as handler error symbol. Help people read code and simplify code.

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
```dart
n := io.Write(x) #err       // 1. err as value

n := io.Write(x) #@ignore   // 2. ignore error

n := io.Write(x) #@done     //  3. return error immediately、

n := io.Write(x) #@wrap("tcp closed: %w") // 4. wrap additional information

n := io.Write(x) #@must     // 5. panic err
```

1. err as value. everything is like before. just error at suffix.
2. ignore error. make it no easier to ignore error than before. Encourage people with additional information. `igmore` 6 char more than other 4 char
3. return error immediately. Many times, Especially in libiary. we just need returen error nothing need to do. For example: [url.parseAuthority](https://github.com/golang/go/blob/master/src/net/url/url.go#L586)
4. wrap additional information. maybe more easy way is  `#@wrap` equal to `#@wrap("io.Wirite err:")`, omit parameter make it easier for people to handle error instead of ignore error.
5. Many third-party libraries have `MustXxx()` and `Xxx()` two sets api. Now we just need only one. Just like `go` keyword, we no longer need sync and async functions.

People should reduce the discussion of compilation details. Focusing on achieving agreement on lang style.

## example

[go/issues/21161](https://github.com/golang/go/issues/21161#issuecomment-390216685)
```dart
func NewClient(...) (*Client, error) {
	var err error

	listener := net.Listen("tcp4", listenAddr) #@done
	defer func() {
		if err != nil {
			listener.Close()
		}
	}()

	conn := ConnectionManager{}.connect(server, tlsConfig) #@done
	defer func() {
		if err != nil {
			conn.Close()
		}
	}()

	if forwardPort == 0 {
		env := environment.GetRuntimeEnvironment() #err
		if err != nil {
			log.Printf("not forwarding because: %v", err)
		} else {
			forwardPort = env.PortToForward() #err
			if err != nil {
				log.Printf("env couldn't provide forward port: %v", err)
			}
		}
	}
	var forwardOut *forwarding.ForwardOut
	if forwardPort != 0 {
		u := url.Parse(fmt.Sprintf("http://127.0.0.1:%d", forwardPort)) #@ignore
		forwardOut = forwarding.NewOut(u)
	}

	client := &Client{...}

	toServer := communicationProtocol.Wrap(conn)
	toServer.Send(&client.serverConfig) #@done
	toServer.Send(&stprotocol.ClientProtocolAck{ClientVersion: Version}) #@done

	client.session = communicationProtocol.FinalProtocol(conn) #@done
	return client, nil
}
```

[url.parseAuthority](https://github.com/golang/go/blob/master/src/net/url/url.go#L586)
``` dart
func parseAuthority(authority string) (user *Userinfo, host string, err error) {
	i := strings.LastIndex(authority, "@")
	if i < 0 {
		host = parseHost(authority) #@done
		return nil, host, nil
	} else {
		host = parseHost(authority[i+1:]) #@done
	}

	userinfo := authority[:i]
	if !validUserinfo(userinfo) {
		return nil, "", errors.New("net/url: invalid userinfo")
	}
	if !strings.Contains(userinfo, ":") {
		userinfo = unescape(userinfo, encodeUserPassword) #@done
		user = User(userinfo)
	} else {
		username, password, _ := strings.Cut(userinfo, ":")
		username = unescape(username, encodeUserPassword) #@done
		password = unescape(password, encodeUserPassword) #@done
		user = UserPassword(username, password)
	}
	return user, host, nil
}
```

## Method chaining
move error to suffix have other benefit that separate return value and error. now we can Method chaining. 
Some people dislike Method Chaining but I like it.

```dart 
func div(a, b float64) (float64, error) {...}
func sum(a, b float64) (float64, error) {...}

func handler() (float64, error){
  res := sum(div(0,7), div(4,5)) #err
  if err != nil{
      log.Error("do calc err: ", err)
      return 0, err 
  }
  return res, nil
}
```

## some link
https://go.googlesource.com/proposal/+/master/design/go2draft-error-handling.md
