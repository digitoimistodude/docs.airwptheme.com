---
description: >-
  This documentation is about how to implement a real time search to Air-light
  based WordPress themes. Written on 5.2.2022 to be used before WordPress plugin
  or Air-light functionality is ready.
---

# Real time search with Vue.js

## Install and set up Vue.js packages

Install necessary packages in **project root folder**:

```bash
npm install vue@2.6.14 vue-loader@15.9.7 vue-template-compiler@2.6.14 --save-dev
```

Run in **theme folder**:

```bash
npm install vue axios --save
```

Open `gulp/webpack.config.dev.js` and make it look like the following (the actual changes to default configs are loading up vue-loader (the first line here) and adding vue-related test, loader, resolve, alias and plugins):

```javascript
module.exports = {
  externals: {
    jquery: 'jQuery' // Available and loaded through WordPress.
  },
  mode: 'development',
  module: {
    rules: [
      {
        test: /.js$/,
        exclude: /node_modules/,
        use: [{
          loader: 'babel-loader',
          options: {
            presets: [
              ['airbnb', {
                targets: {
                  chrome: 50,
                  ie: 11,
                  firefox: 45
                }
              }]
            ]
          }
        }]
      },
      {
        test: /\.vue$/,
        loader: 'vue-loader'
      },
    ]
  },
  resolve: {
    alias: {
      'vue$': 'vue/dist/vue.esm.js' // 'vue/dist/vue.common.js' for webpack 1
    }
  },
  plugins: [
    (compiler) => {
      const VueLoaderPlugin = require('vue-loader/lib/plugin');
      new VueLoaderPlugin().apply(compiler);
  },
  ],
};
```

Next make `gulp/webpack.prod.js` look like this:

```javascript
module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      (compiler) => {
        const TerserPlugin = require('terser-webpack-plugin');
        new TerserPlugin({
        terserOptions: {
          // ecma: 6,
          parse: {},
          compress: {},
          mangle: true, // Note `mangle.properties` is `false` by default.
          module: false,
          toplevel: false,
          nameCache: null,
          ie8: false,
          keep_fnames: false,
          safari10: false,
          format: {
            comments: false,
          },
        },
        extractComments: false,
      }).apply(compiler);
    },
  ]
  },
  externals: {
    jquery: 'jQuery' // Available and loaded through WordPress.
  },
  mode: 'production',
  module: {
    rules: [
      {
        test: /.js$/,
        exclude: /node_modules/,
        use: [{
          loader: 'babel-loader',
          options: {
            presets: [
              ['airbnb', {
                targets: {
                  chrome: 50,
                  ie: 11,
                  firefox: 45
                }
              }]
            ]
          }
        }]
      },
      {
        test: /\.vue$/,
        loader: 'vue-loader'
      },
    ]
  },
  resolve: {
    alias: {
      'vue$': 'vue/dist/vue.esm.js' // 'vue/dist/vue.common.js' for webpack 1
    }
  },
  plugins: [
    (compiler) => {
      const VueLoaderPlugin = require('vue-loader/lib/plugin');
      new VueLoaderPlugin().apply(compiler);
  },
  ],
};
```

Your project should now compile and understand Vue.js when you run `gulp` or `gulp js`.

## Add Vue templates

If you are using VSCode, add following to your settings.json to prevent fileheaders in .vue as it may mess up the template:

```json
"fileheader.ignore": [
  "*.vue",
  "*.txt",
  "*.scss",
  "*.css",
  "*.xml",
  "*.svg",
  "style.css"
],
```

Create folder under theme directory `js/src/apps`. Add subfolder `js/src/apps/search`. Add subfolders `js/src/apps/search/inc` and `js/src/apps/search/views`. Under `inc` folder add `api.js` as following. Again, replace `textdomain` with correct textdomain.

```javascript
import axios from 'axios';

const apiURL = window.textdomain_apiURL;

export const api = axios.create({
  baseURL: `${apiURL}`,
});

/**
 * Parse taxonomies from WP Rest API response
 *
 * @param Array taxonomies Taxonomies
 */
export const parseFilters = (filter) => Object.values(filter).map((filter) => {
  const {
    term_id: id, count, slug, name, taxonomy,
  } = filter;
  return {
    id, count, slug, name, taxonomy, selected: false,
  };
});
```

Add following to `js/src/apps/search/views/search.vue` template. Again, replace `textdomain` with correct textdomain.

```html
<template>
  <div class="search-wrapper" role="search">
    <button type="button" class="search-toggle search-trigger" v-on:click="toggle()">
      <span class="search-label" :class="toggled ? 'hidden' : ''">
        <svg aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" class="search-icon">
          <g fill="none" fill-rule="evenodd" stroke="#393939" stroke-width="2">
            <circle aria-hidden="true" cx="10.5" cy="10.5" r="9.5"/>
            <path stroke-linecap="round" stroke-linejoin="round" d="M18 18l5.401 5.401"/>
          </g>
        </svg><span class="screen-reader-text">Hae</span>
      </span>
      <span class="close-label" :class="! toggled ? 'hidden' : ''">
        <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="currentColor" class="close-icon">
        <path d="M13.46 12L19 17.54V19h-1.46L12 13.46 6.46 19H5v-1.46L10.54 12 5 6.46V5h1.46L12 10.54 17.54 5H19v1.46L13.46 12z"></path>
        </svg><span class="screen-reader-text">Sulje</span>
      </span>
    </button>

    <section class="search-panel" v-if="toggled">

      <div class="search-field search-form">
        <div class="container">
          <h2>{{localization.blockTitle}}</h2>
          <input :placeholder="localization.inputLabel" type="text" name="search-field" id="search-field" class="search-input" ref="search" v-model="searchQuery" v-on:input="search()"/>
        </div>
      </div>

      <div class="result-container" :class="searchStatus">
        <div class="load-animation">
          <span class="circle"></span>
          <span class="circle"></span>
          <span class="circle"></span>
        </div>
        <div class="container">
          <div class="instructions" v-if="!resultCount && searchQuery.length < 3">
            <p>{{localization.instructions}}</p>
          </div>

          <div class="results" v-if="Object.keys(results).length">
            <template v-for="(resultGroup, key) in results">
              <div class="result-group"  :class="key" :key="key" v-if="resultGroup.count">
                <h2>{{resultGroup.title}} <span>({{resultGroup.count}})</span></h2>
                <ul>
                  <li v-for="item in resultGroup.items" :key="item.id">
                    <div v-html="item.html"></div>
                  </li>
                </ul>
              </div>
            </template>
          </div>

          <div class="no-results" v-if="resultCount === 0">
            <p>{{localization.noResults}} <strong>{{searchQuery}}</strong></p>
          </div>

        </div>
      </div>
    </section>

  </div>
</template>

<script>
import {api} from '../inc/api';
export default {
  data: function () {

    return {
      toggled: false,
      searchQuery: '',
      results: {},
      timeout: false,
      localization: window.textdomain_searchLocalization,
      resultCount: -1,
      searchStatus: 'visible',
    }
  },
  mounted() {
    // Add listener for closing the toggled search with esc key
    window.addEventListener('keyup', event => {
      if (! this.toggled) {
        return;
      }
      let isEscape = false;
      if ("key" in event) {
        isEscape = (event.key === "Escape" || event.key === "Esc");
      } else {
        isEscape = (event.keyCode === 27);
      }
      if (! isEscape) {
        return;
      }
      this.toggled = false;
    });
  },

  methods: {
    search() {
      if ( this.searchQuery.length < 3 ) {
        return;
      }
      if (this.timeout) {
        clearTimeout(this.timeout);
      }

      this.timeout = setTimeout(() => {
        this.fetchResults();
        this.searchStatus = 'loading';
      }, 800);
    },
    fetchResults() {

      api
      .get( textdomain_searchLocalization.apiUrl + `textdomain/v1/search/?s=${this.searchQuery}`)
      .then(response => {
        this.results = response.data;

        this.resultsLoaded();

        console.log(response.data);
      })
      .catch(error => {
        console.log(error);
      });
    },
    resultsLoaded() {
      // Let it animate
      this.searchStatus = 'loaded';
      let searchResults = [];
      // Check if we have any results and update status
      for (const key in this.results) {
        if (this.results.hasOwnProperty(key) && this.results[key].hasOwnProperty('items')) {
          searchResults = [
            ...searchResults,
            ...this.results[key].items.map(item => item.id)
          ];
        }
      }

      this.resultCount = searchResults.length;

      setTimeout(() => this.searchStatus = 'visible', 1000);
    },
    toggle() {
      this.toggled = ! this.toggled;

      // Focus when search is opened
      if (this.toggled) {
        this.$nextTick(() => {
          this.$refs.search ? this.$refs.search.focus() : '';
        });
      }
    }
  },
}
</script>
```

Search for `workpower` and `atena` and replace all occurrences with your project textdomain.

Create `js/src/search.js`:

```js
import initSearch from './apps/search/app';

const targetId = 'search';
const target = targetId ? document.getElementById(targetId) : false;

if (target) {
  initSearch(target);
}
```

## Add search related WordPress hooks

Copy the following to `inc/hooks/search-api.php` file. Replace all occurrences of `textdomain` with your project name.

Set up custom post types to this file to match yours, for example in following you have `job` and `person`. Remove them and add yours.

```php
<?php
/**
 * @Author: Timi Wahalahti
 * @Date:   2021-06-11 13:17:44
 * @Last Modified by:   Timi Wahalahti
 * @Last Modified time: 2021-09-15 14:59:59
 *
 * @package textdomain
 */

namespace Air_Light;

// Add our REST API endpoint for search.
function search_endpoint_init() {
  register_rest_route( 'textdomain/v1',
    '/search',
      array(
      'methods'   => 'GET',
      'callback'  => __NAMESPACE__ . '\search_endpoint_callback',
    )
  );
} // end wpfi_rest_api_search_init

// REST API search endpoint callback.
function search_endpoint_callback( $request ) {
  $data = [];
  $current_blog_id = get_current_blog_id();

  // bail if no search param.
  if ( ! isset( $_GET['s'] ) ) { // phpcs:ignore WordPress.Security.NonceVerification
    return $data;
  }

  // used later to prepare post object
  $rest_controller = new \WP_REST_Post_Types_Controller();

  $search_phrase = sanitize_text_field( $_GET['s'] ); // phpcs:ignore WordPress.Security.NonceVerification

  // try to load from cache
  if ( false !== wp_cache_get( $search_phrase, 'textdomainsearch' ) ) {
    return wp_cache_get( $search_phrase, 'textdomainsearch' );
  }

  // Build default args
  $default_args = array(
    'WP_Query' => [
      's'                      => $search_phrase,
      'posts_status'           => 'publish',
      'has_password'           => false,
      'no_found_rows'          => true,
      'cache_results'          => true,
      'update_post_term_cache' => false,
      'posts_per_page'         => 999, // phpcs:ignore WordPress.WP.PostsPerPage
    ],
  );

  $queries = [
    'person' => [
      'type'  => 'WP_Query',
      'title' => ask__( 'Haku: Henkilöt' ),
      'args'  => [
        'post_type' => 'person',
      ],
    ],
    'job' => [
      'type'  => 'WP_Query',
      'title' => ask__( 'Haku: Työpaikat' ),
      'args'  => [
        'post_type' => 'job',
      ],
    ],
    'post' => [
      'type'  => 'WP_Query',
      'title' => ask__( 'Haku: Sisältö' ),
      'args'  => [
        'post_type' => [ 'post', 'page' ],
      ],
    ],
  ];

  foreach ( $queries as $key => $settings ) {
    if ( 'person' === $key ) {
      switch_to_blog( get_main_site_id() );
    }

    $classname = $settings['type'];
    $query = new $classname( array_merge( $default_args[ $classname ], $settings['args'] ) );
    $data[ $key ]['title'] = $settings['title'];

    relevanssi_do_query( $query );

    // Loop results if WP_Query
    if ( 'WP_Query' === $settings['type'] && $query->have_posts() ) {
      $data[ $key ]['count'] = $query->post_count;

      while ( $query->have_posts() ) {
        $query->the_post();

        // add result to response
        $item = $rest_controller->prepare_response_for_collection( $query->post );

        ob_start();
        if ( 'job' === $key ) {
          include get_template_part( 'template-parts/loops', 'job', [ 'job_id' => $item->ID ] );
        } elseif ( 'person' === $key ) {
          include get_template_part( 'template-parts/loops', 'person', [ 'person_id' => $item->ID, 'image' => 'tag' ] );
        } else {
          include get_template_part( 'template-parts/loops', 'post', [ 'post_id' => $item->ID ] );
        }

        $html = ob_get_clean();
        $data[ $key ]['items'][] = [
          'id'    => $item->ID,
          'html'  => $html,
        ];
        continue;

        if ( 'job' === $key ) {
          $item->job_details = get_job_details( $item->ID );
        }

        if ( 'person' === $key ) {
          $item->person_details = get_person_details( $item->ID );
        }

        // remove some un-necssary data
        unset( $item->post_content );
        unset( $item->post_password );
        unset( $item->to_ping );
        unset( $item->pinged );
        unset( $item->post_content_filtered );
        unset( $item->post_mime_type );
        unset( $item->filter );
        unset( $item->menu_order );

        $data[ $key ]['items'][] = $item;
      }
    }

    if ( 'person' === $key ) {
      switch_to_blog( $current_blog_id );
    }
  }

  wp_cache_add( $search_phrase, $data, 'textdomainsearch', HOUR_IN_SECONDS );

  return $data;
} // end search_endpoint_callback
```

In `search-api.php`, define all post types to that file where we should be searching content from, `$queries` example:

```php
$queries = [
  'person' => [
    'type'  => 'WP_Query',
    'title' => ask__( 'Haku: Henkilöt' ),
    'args'  => [
      'post_type' => 'person',
    ],
  ],
  'product' => [
    'type'  => 'WP_Query',
    'title' => ask__( 'Haku: Tuotteet' ),
    'args'  => [
      'post_type' => 'product',
    ],
  ],
  'guide' => [
    'type'  => 'WP_Query',
    'title' => ask__( 'Haku: Oppaat' ),
    'args'  => [
      'post_type' => 'guide',
    ],
  ],
  'webinar' => [
    'type'  => 'WP_Query',
    'title' => ask__( 'Haku: Webinaarit' ),
    'args'  => [
      'post_type' => 'webinar',
    ],
  ],
  'post' => [
    'type'  => 'WP_Query',
    'title' => ask__( 'Haku: Sisältö' ),
    'args'  => [
      'post_type' => [ 'post', 'page' ],
    ],
  ],
];
```

Make sure localizations for titles exist.

In `search-api.php`, add correct template-parts:

```php
if ( 'job' === $key ) {
  include get_template_part( 'template-parts/loops', 'job', [ 'job_id' => $item->ID ] );
} elseif ( 'person' === $key ) {
  include get_template_part( 'template-parts/loops', 'person', [ 'person_id' => $item->ID, 'image' => 'tag' ] );
} else {
  include get_template_part( 'template-parts/loops', 'post', [ 'post_id' => $item->ID ] );
}
```

In `inc/hooks/scripts-styles.php` add the following right after 'styles' is being enqueued. Again, replace `textdomain` with correct textdomain.

```php
// Search application
wp_enqueue_script( 'search',
  get_theme_file_uri( get_asset_file( 'search.js' ) ),
  [],
  filemtime( get_theme_file_path( get_asset_file( 'search.js' ) ) ),
  true
);

wp_localize_script( 'search', 'textdomain_searchLocalization', [
  'apiUrl'        => get_rest_url(),
  'blockTitle'    => ask__( 'Haku: Hae sivustolta' ),
  'inputLabel'    => ask__( 'Haku: Hae' ),
  'noResults'     => ask__( 'Haku: Ei hakutuloksia' ),
  'instructions'  => ask__( 'Haku: Hakuohjeet' ),
] );
```

Add to your `inc/includes/localization.php` (add or remove your custom post type labels):

```php
'Haku: Ei hakutuloksia' => 'Ei hakutuloksia',
'Haku: Hakuohjeet'      => '',
'Haku: Hae sivustolta'  => 'Hae sivustolta',
'Haku: Hae'             => 'Hae',
'Haku: Tuotteet'        => 'Tuotteet',
'Haku: Webinaarit'      => 'Webinaarit',
'Haku: Oppaat'          => 'Oppaat',
'Haku: Sisältö'         => 'Sisältö',
```

Add to your `inc/hooks.php`:

```php
/**
 * Enable search view
 */
add_filter( 'air_helper_disable_views_search', '__return_false' );
add_filter( 'air_helper_disable_views_category', '__return_false' );
add_filter( 'air_helper_disable_views_tag', '__return_false' );

/**
 * REST API related
 */
require get_theme_file_path( 'inc/hooks/search-api.php' );
add_action( 'rest_api_init', __NAMESPACE__ . '\search_endpoint_init' );
```

Lastly, check that all your custom post types that are needed to be found have `show_in_rest` as `true`.

## Add search page template

Use the following `search.php` template. Again, replace all occurrences of `textdomain` with your project name.

```php
<?php
/**
 * The template for displaying search results pages
 *
 * @link https://developer.wordpress.org/themes/basics/template-hierarchy/#search-result
 *
 * @Date:   2019-10-15 12:30:02
 * @Last Modified by:   Timi Wahalahti
 * @Last Modified time: 2021-06-11 14:22:22
 *
 * @package textdomain
 */

namespace Air_Light;

$results = [];

if ( ! empty( $_GET['s'] ) && have_posts() ) {
  while ( have_posts() ) {
    the_post();
    $post_type = get_post_type();

    $results[ $post_type ]['posts'][] = [
      'title'     => (string) get_the_title(),
      'permalink' => (string) get_permalink(),
      'excerpt'   => (string) get_the_excerpt(),
    ];
  }
} wp_reset_postdata();

// Get post type objects for results
foreach ( $results as $slug => $post_type ) {
  $results[ $slug ]['object'] = (object) get_post_type_object( $slug );
  $results[ $slug ]['count']  = (int) count( $results[ $slug ]['posts'] );
}

get_header(); ?>

<main class="site-main">
  <div id="search"></div>
</main>

<?php get_footer();
```

## Add overlay search template

If you want to use overlay search like on [Atena.fi](https://atena.fi), add the following to `header.php` inside site-header element right after `<?php get_template_part( 'template-parts/header/navigation' ); ?>`.

```html
<div id="search"></div>
```

## Add basic styling

Add `sass/views/search.scss`:

```scss
:root {
  --height-site-header: 100px;
}

// Search
.block-search,
.block-search-results {
  background-color: var(--color-white);
}

.search-form {
  input,
  label {
    width: 100%;
  }

  .container {
    border-bottom: 1px solid var(--color-hr-grey);

    h2 {
      @media (min-width: $container-ipad) {
        margin-bottom: auto;
        margin-top: auto;
      }
    }

    .search-input {
      background-image: url('../../svg/search-magnifying.svg');
      background-position: left 23px center;
      background-repeat: no-repeat;
      border-radius: 0;
      border-width: 1px;
      font-weight: var(--font-weight-regular);
      padding-bottom: 2.5rem;
      padding-left: 7.5rem;
      padding-top: 2.5rem;
    }

    @media (min-width: $container-ipad) {
      column-gap: 2rem;
      display: grid;
      grid-template-columns: 1fr 3fr;
      padding-top: 9rem;
    }
  }
}

.search-wrapper .results {
  @include grid(3, $gutter_x: 80px, $gutter_y: 0);

  h2 span {
    font-size: var(--font-size-20);
  }

  li {
    font-size: var(--font-size-16);
    margin: 0 0 4rem;
    padding: 0;
  }

  h3 {
    font-size: var(--font-size-18);
  }

  a {
    color: var(--color-black);
  }

  // We already have link in title and structure here cannot be changed
  // so hiding this is OK.
  // stylelint-disable-next-line a11y/no-display-none
  .global-link {
    display: none;
  }

  .result-group ul {
    list-style: none;
    margin: 0;
    padding: 0;
  }

  // stylelint-disable-next-line selector-max-combinators, selector-max-compound-selectors
  .result-group h3 a:hover,
  .result-group h3 a:focus {
    text-decoration: underline;
  }

  // stylelint-disable-next-line selector-max-combinators, selector-max-compound-selectors
  .result-group.job li > div {
    // @include job();
    align-items: flex-start;
    margin-bottom: 0;

    // stylelint-disable-next-line selector-max-combinators, selector-max-compound-selectors
    svg {
      margin-right: 2.5rem;
      margin-top: 1rem;
    }
  }

  @media (max-width: 1500px) {
    @include grid(3, $gutter_x: 28px, $gutter_y: 0);
  }

  @media (max-width: 1300px) {
    @include grid(2, $gutter_x: 28px, $gutter_y: 28px);
  }

  @media (max-width: 765px) {
    @include grid(1, $gutter_x: 28px, $gutter_y: 28px);
  }
}

.search-panel {
  background-color: var(--color-white);
  height: calc(100vh - var(--height-site-header));
  left: 0;
  overflow-x: hidden;
  overflow-y: auto;
  padding: 0;
  position: absolute;
  top: 92px;
  width: 100%;
  z-index: 999;

  .result-container {
    min-height: 20rem;
    position: relative;
  }

  .load-animation {
    align-items: flex-start;
    background-color: var(--color-white);
    display: flex;
    height: calc(100% - 8rem);
    justify-content: center;
    opacity: 0;
    padding-top: 8rem;
    pointer-events: none;
    position: absolute;
    transition: opacity .4s ease-in-out;
    width: 100%;

    .circle {
      animation-duration: .5s;
      animation-iteration-count: infinite;
      animation-name: jump;
      animation-timing-function: ease-in-out;
      background-color: var(--color-main);
      border-radius: 50%;
      height: 10px;
      margin: 2px;
      width: 10px;
    }

    .circle:nth-of-type(2) {
      animation-delay: .1s;
    }

    .circle:nth-of-type(3) {
      animation-delay: .2s;
    }

    @keyframes jump {
      0%,
      100% {
        transform: translateY(0);
      }

      50% {
        transform: translateY(-10px);
      }
    }
  }

  .visible .load-animation {
    opacity: 0;
    z-index: 0;
  }

  .loaded .load-animation {
    z-index: 10;
  }

  .loading .load-animation {
    opacity: 1;
    z-index: 10;
  }
}

button.search-toggle {
  // stylelint-disable-next-line scss/at-extend-no-missing-placeholder
  @extend .menu-item;
  align-items: center;
  background-color: transparent;
  background-image: none;
  border: 0;
  display: inline-flex;
  font-weight: 600;
  padding: 0;
  transition: all $transition-duration;

  &:hover,
  &:focus {
    cursor: pointer;
    opacity: .5;
  }
}
```

Add `svg/search-magnifying.svg`:

```html
<svg aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" class="search-icon"><g fill="none" fill-rule="evenodd" stroke="#393939" stroke-width="2"><circle aria-hidden="true" cx="10.5" cy="10.5" r="9.5"></circle><path stroke-linecap="round" stroke-linejoin="round" d="M18 18l5.401 5.401"></path></g></svg>
```

## Customize the rest of the search

Check that all content displays correctly. Check also the regular search page: https://textdomain.test/?s=something

That's all!
