# WordPress Theming Notes
A collection of notes made while experimenting with Wordpress theming.

## Helpful Links
* [WordPress Documentation](https://developer.wordpress.org/)
* [Coding Standards Handbook](https://developer.wordpress.org/coding-standards/)
* [WordPress Theme Handbook](https://developer.wordpress.org/themes/)
* [WordPress Code Reference](https://developer.wordpress.org/reference/)
* [Template Hierarchy](https://developer.wordpress.org/themes/basics/template-hierarchy/)
* [Template Hierarchy - Visual](https://wphierarchy.com/)

# Basic Concepts
* **Theme** - In WordPress, a theme is a collection of templates and stylesheets used to define the appearance and display of a WordPress powered website.
* **Starter Theme** - A WordPress starter theme is a blank theme with minimum design, and a basic or absolutely no layout. These themes usually come with the most commonly used templates in a WordPress theme.
* **Parent Theme** - A parent theme in WordPress is a theme that is declared parent by a another theme, child theme. This feature in WordPress allows theme designers and developers to take advantage of a larger and robust WordPress themes and make modifications to those themes by creating child themes.
* **Child Theme** - A child theme in WordPress is a sub theme that inherits all the functionality, features, and style of its parent theme.
* **Child vs Starter Theme** - Create a child theme when you need to edit an exisiting theme & you want to save the changes on (parent) theme update. Use a starter theme when you want to create a new theme with some of the features, structure & code already set up.

# Plugins
A list of helpful plugins to use.
## Development
* [Theme Check](https://wordpress.org/plugins/theme-check/) - The theme check plugin is an easy way to test your theme and make sure it’s up to spec with the latest theme review standards
* [Query Monitor](https://wordpress.org/plugins/query-monitor/) - Query Monitor is the developer tools panel for WordPress. It enables debugging of database queries, PHP errors, hooks and actions, block editor blocks, enqueued scripts and stylesheets, HTTP API calls, and more.
* [WP Reset](https://wordpress.org/plugins/wp-reset/) - WP Reset quickly resets the site’s database to the default installation values without modifying any files. It deletes all customizations and content, or just chosen parts like theme settings.
## Post Types & Fields
* [Custom Post Type UI](https://wordpress.org/plugins/custom-post-type-ui/) - Custom Post Type UI provides an easy to use interface for registering and managing custom post types and taxonomies for your website.
* [Advanced Custom Fields](https://wordpress.org/plugins/advanced-custom-fields/)- This field builder allows you to quickly and easily add fields to WP edit screens with only the click of a few buttons!
## Child Theme
* [Child Theme Generator](https://wordpress.org/plugins/child-theme-generator/) - This plugin will generate a child theme in few steps, quickly and safely, it will not slow down your website or spam your database.

# Tools
* [Generate WP](https://generatewp.com/) - GenerateWP is a set of tools originally designed for WordPress developers to help them decrease development time by generating various snippets of code.

# Optional CSS Framework(s) Utilization
* [Tailwind CSS](https://tailwindcss.com/) with WordPress - https://paulund.co.uk/using-tailwind-css-in-your-wordpress-theme

# Findings While Reverse Engineering an Existing Theme
Following are some observations while studying the WP Bootstrap 4 theme by [TwoPoints](https://bootstrap-wp.com):
* `/template-parts` was used to store post-format-specific templates such as content.php, content-post.php, content-none.php.
* `/page-templates` is was used to store page layouts
* `get_template_part( 'template-parts/content', get_post_format() );` was used to load content-[post-format].php within the loop in index.php and such places.
* [`esc_url()`](https://developer.wordpress.org/reference/functions/esc_url/), [`esc_html_e()`](https://developer.wordpress.org/reference/functions/esc_html_e/), [`esc_attr()`](https://developer.wordpress.org/reference/) & similar functions have been used widely.
* Comments were handled using [`comments_open()`](https://developer.wordpress.org/reference/functions/comments_open/), [`get_comments_number()`](https://developer.wordpress.org/reference/functions/get_comments_number/),[`comments_template()`](https://developer.wordpress.org/reference/functions/comments_template/).
* [`blog_info()`](https://developer.wordpress.org/reference/functions/bloginfo/) is utilized where applicable like for brand name & such.
* [`wp_nav_menu()`](https://developer.wordpress.org/reference/functions/wp_nav_menu/) was used for menu, which seems to rely on a *walker*.
* [`is_active_sidebar()`](https://developer.wordpress.org/reference/functions/is_active_sidebar/) & [`dynamic_sidebar()`](https://developer.wordpress.org/reference/functions/dynamic_sidebar/) were used for displaying the sidebar widgets.
* [`the_posts_navigation()`](https://developer.wordpress.org/reference/functions/the_posts_navigation/) was used for displaying pagination links.
## functions.php
* A function_exists() check is performed before defining any custom function.
* [add_action()](https://developer.wordpress.org/reference/functions/add_action/) hook has the beed used to call each of the custom functions.
* `functions.php` was utilized to perform these major tasks: setup, initialize widget i.e register sidebars, add styles and scripts, add editor styles, implement some custom features.
### Setup
* [`load_theme_textdomain()`](https://developer.wordpress.org/reference/functions/load_textdomain/) - Make theme available for translation.
* add_theme_support( 'automatic-feed-links' ) - Add default posts and comments RSS feed links to head.
* add_theme_support( 'title-tag' ) - Let WordPress manage the document title.
* add_theme_support( 'post-thumbnails' ) - Enable support for Post Thumbnails on posts and pages.
* add_theme_support( 'post-formats', array( 'gallery', 'video', 'audio', 'status', 'quote', 'link' ) ) - Enable post formats
* add_theme_support( 'woocommerce' ) - Enable support for woocommerce.
* [`register_nav_menus()`](https://developer.wordpress.org/reference/functions/register_nav_menus/) - Registers navigation menu locations for theme.
* [`add_theme_support( 'custom-logo' )`](https://developer.wordpress.org/reference/functions/add_theme_support/) - Add support for core custom logo.
### Initialize Widgets i.e Register Sidebars
The following snippet was used (within a custom setup function) to register sidebars (here sidebar-1)
```php
register_sidebar( array(
      'name'          => esc_html__( 'Sidebar', 'wp-bootstrap-4' ),
      'id'            => 'sidebar-1',
      'description'   => esc_html__( 'Add widgets here.', 'wp-bootstrap-4' ),
      'before_widget' => '<section id="%1$s" class="widget border-bottom %2$s">',
      'after_widget'  => '</section>',
      'before_title'  => '<h5 class="widget-title h6">',
      'after_title'   => '</h5>',
) );
```
### Add Styles and Scripts
The following snippet was used:
```php
function wp_bootstrap_4_scripts() {
	wp_enqueue_style( 'open-iconic-bootstrap', get_template_directory_uri() . '/assets/css/open-iconic-bootstrap.css', array(), 'v4.0.0', 'all' );
	wp_enqueue_style( 'bootstrap-4', get_template_directory_uri() . '/assets/css/bootstrap.css', array(), 'v4.0.0', 'all' );
	wp_enqueue_style( 'wp-bootstrap-4-style', get_stylesheet_uri(), array(), '1.0.2', 'all' );

	wp_enqueue_script( 'bootstrap-4-js', get_template_directory_uri() . '/assets/js/bootstrap.js', array('jquery'), 'v4.0.0', true );

	if ( is_singular() && comments_open() && get_option( 'thread_comments' ) ) {
		wp_enqueue_script( 'comment-reply' );
	}
}
add_action( 'wp_enqueue_scripts', 'wp_bootstrap_4_scripts' );
```
### Add Editor Styles
The following snippet was used:
```php
function wp_bootstrap_4_add_editor_styles() {
    add_editor_style( 'editor-style.css' );
}
add_action( 'admin_init', 'wp_bootstrap_4_add_editor_styles' );
```
* `editor-style.css` was at theme root.

# Findings While Building a Theme with [Understrap](https://understrap.com/)
Understrap combines the Underscores starter theme (by Automattic) and the mobile-first, responsive grid framework Bootstrap 4 (by Twitter) into a perfect open source foundation for your next WordPress theme project.
* The [Understrap Documentation](https://understrap.github.io/) only consists of information on getting started with the build system, surprisingly there's not much about anything else such as structure.
* **Styles** - Major style files are located at `/sass/theme/_theme.scss` & `/sass/understrap/understrap.scss`.
* **Page Templates** - Located in `/page-templates`, they are: blank, empty, left, right, both and full-width.
* **Loop Templates / Template Parts** - blank, empty, none, page, single, content (default).
* **Sidebar and Global Templates** - Global templates (`/global-templates`) perfrom check and include sidebar templates (`sidebar-templates`).
* **Functions** - `functions.php` includes files from `/inc`, while there is a good seperation of concerns in these, they are not that atomic.
