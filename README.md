# Mnemosyne

Mnemosyne is a multilayer cache which can be configured to use various configurations of [Redis](https://redis.io/) and/or [BigCahce](https://github.com/allegro/bigcache) (an in-memory caching package) with minimum configuration out of the box.

## Getting Started

### Installing

```console
go get -u git.cafebazaar.ir/bazaar/search/mnemosyne
```


### Initializing a Manager and Selecting a Cache Instance

```go
mnemosyneManager := mnemosyne.NewMnemosyne(config, epimetheus)
cacheInstance := mnemosyneManager.select("result-cache")
```

### Working with a cacheInstance
```go
  cacheInstance.Set(context, key, value)
  var myCachedData myType
  err := cacheInstance.Get(context, key, &myCachedData)
  // cache miss is also an Error
```

## Configuration

Mnemosyne uses Viper as it's config engine. Template of each cache instance includes the list of the layers' names (in order of precedence) followed by configuration for each layer.
here's an example: 
```yaml
cache:
  my-result-cache: # arbitary name for the cache instance
    soft-ttl: 2h 
    layers:   # Arbitary names for each cache layer
      - result-memory
      - result-gaurdian
    result-memory:
      type: memory
      max-memory: 512
      ttl: 2h
      amnesia: 100
      compression: true
    result-gaurdian:
      type: gaurdian
      address: "localhost:6379"
      slaves:
        - "localhost:6379"
        - "localhost:6379"
      db: 2
      ttl: 24h
      amnesia: 100
      compression: true
  my-user-cache:
    soft-ttl: 2h
    layers:
      - user-memory
      - user-redis
    user-memory:
      type: memory
      max-memory: 512
      ttl: 2h
      amnesia: 0
      compression: true
    user-gaurdian:
      type: gaurdian
      address: "localhost:6379"
      slaves:
        - "localhost:6379"
        - "localhost:6379"
      db: 4
      ttl: 24h
      amnesia: 0
      compression: true
```

`soft-ttl` is an instance-wide TTL which when expired will **NOT** remove the data from the instance, but warns that the data is old

Each cache layer can be of types `redis`, `gaurdian`, `memory` or `tiny`. `redis` is used for a single node Redis server, `gaurdian` is used for a master-slave Redis cluster configuration, `memory` uses the BigCache library to provide an efficient and fast in-memory cache, `tiny` uses the native sync.map data structure to store smaller cache values in memory (used for low-write caches).
Note: all of the cache types are sync-safe, meaning they can be safely used from simultaneously running goroutines.

#### Common layer configs:

`amnesia` is a stochastic fall-through mechanism which allows for a higher layer to be updated from a lower layer by the way of an artificial cache-miss, 
a 0 amnesia means that the layers will never miss a data that they actually have, a 10 amnesia means when a key is present in the cache, 90% of the time it is returned but 10% of the time it is ignored and is treated as a cache-miss. a 100 amnesia effectively turns the layer off. (Default: 0)

`compression` is whther the data is compressed before being put into the cache memory. Currently only Zlib compression is supported. (Default: false)

`ttl` is the hard Time To Live for the data in this particular layer, after which the data is expired and is expected to be removed.

#### Type-spesific layer configs:

`db` [`redis` - `gaurdian`] is the Redis DB number to be used. (Default:0)
`idle-timeout` [`redis` - `gaurdian`] is the timeout for idle connections to the Redis Server (see Redis documentation) (Default:0 - no timeout)
`address` [`redis` - `gaurdian`] is the Redis Server's Address (the master's address in case of a cluster)
`slaves` [`gaurdian`] is a **list** of Redis servers addresses pertaining to the slave nodes.
`max-memory` [`memory`] is the maximum amount of system memory which can be used by this particular layer.


## Documentation

Documents are available at [https://godoc.org/git.cafebazaar.ir/bazaar/search/mnemosyne](https://godoc.org/git.cafebazaar.ir/bazaar/search/mnemosyne)

## Built With

* [Redis Go clinet](https://github.com/go-redis/redis) - Redis Client for Golang
* [BigCahce](https://github.com/allegro/bigcache) - An In-Memory cache library

## Contributing

Please read [CONTRIBUTING.md](https://git.cafebazaar.ir/bazaar/search/mnemosyne/blob/master/CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://git.cafebazaar.ir/bazaar/search/mnemosyne/tags). 

## Roadmap
    - Improve documentation
    - Add tests

## Authors

* **Ramtin Rostami** - *Initial work* - [rrostami](https://github.com/rrostami)
* **Pedram Teymoori** - *Initial work* - [pedramteymoori](https://github.com/pedramteymoori)
* **Parsa abdollahi** - *Initial work* - []()

See also the list of [contributors](https://git.cafebazaar.ir/bazaar/search/mnemosyne/-/graphs/master) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* [Sepehr nematollahi](https://www.behance.net/sseeppeehhrr) Epimetheus typography designer

Made with <span class="heart">❤</span> in Cafebazaar search