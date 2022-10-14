# Caching

All ACF blocks that are rendered with default functionality by using `render_acf_block` callback are cached. By default, WordPress Core object cache is not that useful so we recommend replacing it with something like [Object Cache Pro](https://objectcache.pro/) -plugin that uses Redis.

### Cache lifetime & invalidation

Blocks are cached for one hour. Change this with a `air_acf_block_cache_lifetime` filter. For example, like this.

```php
add_filter( 'air_acf_block_cache_lifetime', function( $lifetime, $block_slug, $post_id ) {
  return MINUTE_IN_SECONDS * 30;
}, 10, 3 );
```

There is no warmup for block caching, so for the first visitor that visits a page, blocks are rendered with PHP at the request. For the next visitors, cached block HTML will be used. When the cache for a specific block gets old, again next visitor will trigger the generation of cached HTML.

Blocks are cached with a key that contains a hash of all block data, so every time that block contents are changed the cache also invalidates automatically. If the block uses, for example, data that comes from meta fields on the page, then it's wise to add the page updated timestamp to the cache key. That way block cache invalidates automatically every time page is updated. This can be achieved with the following example.

```php
add_filter( 'air_acf_block_cache_key', function( $cache_key, $block_slug, $post_id ) {
  $cache_key .= '|' . get_post_timestamp( $post_id, 'modified' );
  return $cache_key;
}, 10, 3 );
```

### Disable caching

Block caching is active only in production environments.

Disable block caching per block in the `THEME_SETTINGS` array by adding a `'prevent_cache' => true,` to the block registration.

There is also `air_acf_block_maybe_enable_cache` filter available to disable the caching for all blocks or per block.

```php
add_filter( 'air_acf_block_maybe_enable_cache', function( $cache, $block_slug ) {
  return false;
}, 10, 2 );
```

