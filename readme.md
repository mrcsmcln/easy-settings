# Easy Settings

 1. [Installation](#installation)
     1. [Standalone Plugin](#standalone-plugin)
     2. [Part of a Theme](#part-of-a-theme)
 2. [Structure](#structure)
 3. [Naming](#naming)
 4. [Creating Pages](#creating-pages)
 5. [Creating Sections](#creating-sections)
 6. [Creating Fields](#creating-fields)
 7. [Supported Submenus](#supported-submenus)
 8. [Configuration](#configuration)
     1. [Configuring Pages](#configuring-pages)
     1. [Configuring sections](#configuring-sections)
     1. [Configuring fields](#configuring-fields)
 9. [Helper Method and Properties](#helper-method-and-properties)

## Installation

First, you will need to decide whether this will be a standalone plugin or part of a theme.

### Standalone Plugin
1. Move `easy-settings/` to `wp-content/plugins`.
2. In the Administration Screen, choose Plugins.
3. Find Easy Settings and click Activate.

### Part of a Theme
1. Move `easy-settings/` to `wp-content/themes`.
2. Open `easy-settings.php`.
3. Find `const TYPE = 'plugin';` and change it to `const TYPE = 'theme';`.
4. Add `load_template( 'easy-settings/easy-settings.php' );` to `functions.php`.

That's it! You're all set to start creating options pages.

## Structure

Creating options pages with Easy Settings is easy. When you create directories and files, Easy Settings parses them and adds the appropriate pages, sections, and fields. Consider the following directory:

```
easy-settings/
    theme-pages/
        my-theme-page/
            my-theme-section/
                index.php
                my-theme-field.php
            index.php
    easy-settings.php
    readme.md
```

Here's how this particular implementation works:

 1. Every subdirectory of `easy-settings` refers to a submenu.
 2. Every subdirectory of `theme-pages` refers to a page within the `Appearance` submenu.
 3. Every subdirectory of `my-theme-page` refers to a section on the `my-theme-page` page.
 4. `index.php` in `my-theme-page` is the template for `my-theme-page`.
 5. `index.php` in `my-theme-section` is the template for `my-theme-section`.
 6. Every other file in `my-theme-section` refers to a field in the `my-theme-section` section.
 
That being said, we can make these rules a bit more abstract:

 1. First-level directories refer to submenus. See [Supported Submenus](#supported-submenus) for more information.
 2. Second-level directories refer to submenu pages.
 3. Third-level directories refer to settings sections.
 4. `index.php` files in second-level directories refer to submenu page templates.
 5. 'index.php` files in third-level directories refer to settings section templates.
 6. Every other file refers to a settings field.
 
## Naming

By default, Easy Settings uses directory and file names to determine the titles of your pages, sections, and fields, using the following algorithm:

 1. Hyphen-minuses (`-`) and underscores (`_`) are replaced by spaces (`&nbsp`).
 2. The first letter of every word is capitalized.

While this algorithm works most of the time, it will fail for titles that include words whose first letter should not be capitalized—"and", "or", etc. To modify the titles of your pages, sections, and fields, you must provide details in their template files. See below for more details.

## Creating Pages

To create a page, you must first create a submenu directory in the Easy Settings root directory (see [Supported Submenus](#supported-submenus) for more information). Let's call our submenu directory `theme-pages`, which will be used to create options pages within the Appearance submenu:

```
easy-settings/
    theme-pages/
```

Next, create your page directory within the submenu directory you just created. We'll call our page directory `my-theme-page`:

```
easy-settings/
    theme-pages/
        my-theme-page/
```

By default, a second-level directory is all you need to get started with your page. However, if you'd like to do some advanced configuration, see [Configuring Pages](#configuring-pages) for more information.

## Creating Sections

To create a section, first make sure you have created the page where your section will be. For more information on this process, check out [Creating Pages](#creating-pages).

Following from the previous section, let's create a section directory. We'll call it `my-theme-section`:

```
easy-settings/
    theme-pages/
        my-theme-page/
            my-theme-section/
```

By default, a third-level directory is all you need to get started with your section. However, if you'd like to do some advanced configuration, see [Configuring Sections](#configuring-sections) for more information.

## Creating Fields

To create a field, first make sure you have created the section where your field well be. For more information on this process, check out [Creating Sections](#creating-sections).

Following from the previous section, let's create a field file. We'll call it `my-theme-field.php`:

```
easy-settings/
    theme-pages/
        my-theme-page/
            my-theme-section/
                my-theme-field.php
```

Note: field files cannot be called `index.php`.

If you leave the field file empty, the default field template will be used. However, if you'd like to do some advanced configuration, see [Configuring Fields](#configuring-fields) for more information.

## Supported Submenus

At the moment, Easy Settings works only with submenu pages, not top-level menu pages. This is to reduce clutter on the Administration Screen.

Simply make a directory in the Easy Settings root directory with a name that corresponds to one of the following submenus:

  - Dashboard: `dashboard-pages`
  - Posts: `posts-pages`
  - Media: `media-pages`
  - Comments: `comments-pages`
  - Appearance: `theme-pages`
  - Plugins: `plugins-pages`
  - Users: `users-pages`
  - Tools: `management-pages`
  - Settings: `options-pages`
  - Settings in the Network Admin pages: `settings-pages`

Easy Settings also supports custom post types: `$type-pages`. Be sure to replace `$type` with your custom post type.

## Configuration

Configuring is easy and straightforward.

### Configuring Pages

To configure a page, you must first have an `index.php` file in its directory. This file will allow you to set various options as well as provide a template for your page. Consider the following sample page configuration file, complete with all the allowed headers:

```
<?php
/*
 * Page Title: My Theme Page
 * Menu Title: My Theme Page
 * Capability: edit_theme_options
 * Sanitize Callback: my_theme_page_sanitize_callback
 */
?>
<div class="wrap">
	<h2><?php echo $this->format_title( $this->current_page ); ?></h2>
	<form action="options.php" method="post"><?php
		settings_fields( $this->current_option );
		do_settings_sections( 'mjm-' . $this->current_page );
		submit_button();
	?></form>
</div>
```
Note: if you provide this file, it will be used for the template, so be sure to supply one.
Note: you must define a sanitize callback function in your theme or plugin if you plan to use this parameter. For example:

```
function my_theme_page_sanitize_callback( $input ) {
    foreach ( $input as &$value ) {
        $value = sanitize_text_field( $value );
    }
    unset( $value );
    return $input;
}
```

### Configuring Sections

Configuring a section is much like configuring a page—you need to have an `index.php` file in its directory and you set your options and template there. Here is a sample section configuration file, complete with the only allowed header:

```
<?php
/*
 * Title: My Theme Section
 */
?>
<p>This is my theme's settings section.</p>
```

These files are usually pretty simple, as you can see.

### Configuring Fields

Configuring a field is a bit different from configuring a page or section, because you aren't dealing with a directory. To configure a field, just use the file you created to refer to that field. If you leave the file completely blank, the default options will be used. Otherwise, the content you put will allow you to set options and the template for the field. Here's a sample field configuration file:

```
<?php
/*
 * Title: My Theme Field
 * Label For: my-theme-field
 */
?>
<input
    type="text"
    id="<?php echo esc_attr( $this->current_field ); ?>"
    name="<?php echo esc_attr( "$this->current_option[$this->current_field]" ); ?>"
    value="<?php echo isset( $this->fields[ $this->current_field ] ) ?
        esc_attr( $this->fields[ $this->current_field ] ) : ''; ?>"
>
```

Note: only use the Label For option if you plan on setting the ID of your field to anything but `esc_attr( $this->current_field )`. In that case, simply set Label For to the new ID.

### Helper Method and Properties

Use the following method and properties to help you with your templates.

  - `$this->format_title( $title )`
  - `$this->current_submenu`
  - `$this->current_page`
  - `$this->current_option`
  - `$this->current_section`
  - `$this->current_field`
  - `$this->fields`
