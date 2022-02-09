# Query Monitor support

ACF block rendering uses [Query Monitor](https://github.com/johnbillion/query-monitor) for logging and to help to debug.

Following log messages are in place:

* When block output starts
* If block is served from cache (includes the cache key)
* If block is saved to cache (includes the cache key)
* If block template file could not be located
