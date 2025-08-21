```Go
var DefaultTransport RoundTripper = &Transport{
        ... 
  MaxIdleConns:          100,
  IdleConnTimeout:       90 * time.Second,
        ... 
}

// DefaultMaxIdleConnsPerHost is the default value of Transport's
// MaxIdleConnsPerHost.
const DefaultMaxIdleConnsPerHost = 2
```

- The `MaxIdleConns: 100` setting sets the size of the connection pool to 100 connections, but _with one major caveat_: this is on a per-host basis. See the comments on the `DefaultMaxIdleConnsPerHost` below for more details on the implications of this.
- The `IdleConnTimeout` is set to 90 seconds, meaning that after a connection stays in the pool and is unused for 90 seconds, it will be removed from the pool and closed.
- The `DefaultMaxIdleConnsPerHost = 2` setting below it. What this means is that even though the entire connection pool is set to 100, there is a _per-host_ cap of only 2 connections!

  

netstat -n | grep -i 8080 | grep -i time_wait | wc -l