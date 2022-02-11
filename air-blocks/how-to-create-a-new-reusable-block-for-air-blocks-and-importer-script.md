# How to create a new reusable block for air-blocks and importer script

This tutorial will show you how to create a new block with:

* PHP block template assets
* SCSS assets
* JS assets
* SVG assets
* Block importer script to be used with `newblock`

### 1. Create a block php file

{% hint style="info" %}
**Please note:** This can be now done with `newblock` command.
{% endhint %}

First you'll have to create a new block template to _template-parts/blocks/your-block.php_:

{% code title="template-parts/blocks/example.php" %}
```php
<?php
/**
 * The template for example
 *
 * Description of your block.
 *
 * @Author:		Roni Laukkarinen
 * @Date:   		2022-02-10 12:28:36
 * @Last Modified by:   Roni Laukkarinen
 * @Last Modified time: 2022-02-10 12:28:36
 *
 * @package airblocks
 * @link https://developer.wordpress.org/themes/basics/template-files/#template-partials
 */

namespace Air_Light;

if ( empty( $title ) ) {
  maybe_show_error_block( 'A title is required' );
  return;
}
?>

<section class="block block-example">
  <div class="container">
   <!-- Your content -->
  </div>
</section>
```
{% endcode %}

### 2. Create styles for the block

Styles will be located under _sass/gutenberg/blocks/_:

{% code title="sass/gutenberg/blocks/_example.scss" %}
```scss
.block-example {
  // Your block styles here
}
```
{% endcode %}

Then, import the block file to _sass/gutenberg/\_blocks.scss_:

{% code title="sass/gutenberg/_blocks.scss" %}
```scss
@import 'gutenberg/blocks/example';
```
{% endcode %}

Compile styles either while gulp is up and running or with a single command:

```bash
gulp devstyles && gulp prodstyles
```

### 3. Register the block in functions

Next you'll need to open functions.php and search:

{% code title="functions.php" %}
```php
'acf_blocks' => [
```
{% endcode %}

After this part, add:

{% code title="functions.php" %}
```php
[
  'name' => 'example',
  'title' => 'Example',
],
```
{% endcode %}

### 4. Create custom fields for the block

Go to wp-admin, **Custom fields > Add New**, add fields, select in **Location** part: **Show this field group if** _Block_ _is equal to_ _Example_. After this, you will be able to add the block in pages.

### 5. Create importer script for the block

Use the following template to add sh file under _bin/blocks/._ Remember to add New files/Dependencies as comments first to clarify things before additions.

{% code title="bin/blocks/example.sh" %}
```bash
#!/bin/bash
# @Author: Roni Laukkarinen
# @Date:   2022-02-10 10:44:02
# @Last Modified by:   Roni Laukkarinen
# @Last Modified time: 2022-02-10 12:23:18

# // New files/Dependencies (this file will install them)::
# // ├── sass/gutenberg/blocks/_example.scss (automatic from get-block.sh)
# // ├── node_modules/example (automatic from theme npm)
# // ├── sass/features/_slick.scss
# // └── svg/block-icons/example.svg (automatic from get-block.sh)

# // Changes to files/folders:
# // ├── sass/gutenberg/_blocks.scss (automatic from get-block.sh)
# // ├── js/src/front-end.js
# // ├── js/src/gutenberg-editor.js
# // ├── acf-json/
# // └── functions.php

# Block specific variables
export BLOCK_ACF_JSON_FILE="XXXXXX.json"
export BLOCK_ACF_JSON_PATH="${AIRBLOCKS_THEME_PATH}/acf-json/${BLOCK_ACF_JSON_FILE}"

# SCSS features
cp -nv ${AIRBLOCKS_THEME_PATH}/sass/features/XXXXXX.scss ${PROJECT_THEME_PATH}/sass/features/

# SCSS style component dependencies
cp -nv ${AIRBLOCKS_THEME_PATH}/sass/components/_XXXXXX.scss ${PROJECT_THEME_PATH}/sass/components/

# JavaScript dependencies
cp -nv ${AIRBLOCKS_THEME_PATH}/js/src/modules/XXXXXX.js ${PROJECT_THEME_PATH}/js/src/modules/

# JS direct replaces
LC_ALL=C sed -i '' -e "s;\/\/ import slick from \'slick-carousel\'\;;import slick from \'slick-carousel\'\;;" ${PROJECT_THEME_PATH}/js/src/front-end.js

# Import js modules right after the last default js module in the front-end.js file
sed -e "/\import \'what-input\'\;/a\\
import './modules/XXXXXX';" < ${PROJECT_THEME_PATH}/js/src/front-end.js > ${PROJECT_THEME_PATH}/js/src/front-end-with-changes.js
rm ${PROJECT_THEME_PATH}/js/src/front-end.js
mv ${PROJECT_THEME_PATH}/js/src/front-end-with-changes.js ${PROJECT_THEME_PATH}/js/src/front-end.js

# Import js modules right after the last default js module in the gutenberg-editor.js file
sed -e "/\import\/no-unresolved \*\//a\\
import slick from 'slick-carousel';" < ${PROJECT_THEME_PATH}/js/src/gutenberg-editor.js > ${PROJECT_THEME_PATH}/js/src/gutenberg-editor-with-changes.js
rm ${PROJECT_THEME_PATH}/js/src/gutenberg-editor.js
mv ${PROJECT_THEME_PATH}/js/src/gutenberg-editor-with-changes.js ${PROJECT_THEME_PATH}/js/src/gutenberg-editor.js

# Other SVG icons needed by this block
cp -nv ${AIRBLOCKS_THEME_PATH}/svg/XXXXXX.svg ${PROJECT_THEME_PATH}/svg/

# Register ACF block in functions.php
# Please note: The title of the block will be translated in localization.sh if en is selected
sed -e "/\'acf_blocks\' \=\> \[/a\\
      [|\
       'name' => 'example',|\
       'title' => 'Example',|\
      ],\\" < ${PROJECT_THEME_PATH}/functions.php | tr '|' '\n' > ${PROJECT_THEME_PATH}/tmpfile

```
{% endcode %}

Add translations to localization.sh if needed. Remember to escape the special characters.

Commit the changes. Add the changes to CHANGELOG.md under \[Unreleased] tag. Push the changes After this your block should be reusable when running the block importer script:

```bash
newblock
```
