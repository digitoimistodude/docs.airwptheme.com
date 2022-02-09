# Caching

All ACF blocks that are rendered with default functionality by using `render_acf_block` callback are cached. By default, WordPress Core object cache is not that useful so we recommend replacing it with something like [Object Cache Pro](https://objectcache.pro) -plugin that uses Redis.

### Cache lifetime & invalidation

Blocks are cached for one hour. Change this with `air_acf_block_cache_lifetime` filter.

There is no warmup for block caching, so for the first visitor that visits a page, blocks are rendered with PHP at the request. For the next visitors, cached block HTML will be used. When the cache for a specific block gets old, again next visitor will trigger the generation of cached HTML.

### Disable caching

### Query Monitor support

