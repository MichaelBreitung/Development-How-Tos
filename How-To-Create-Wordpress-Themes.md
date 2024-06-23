

# How To Create Custom WordPress Themes

This guide gives a short introduction on how to create custom WordPress themes. 

## Template Files

### Mandatory

A WordPress theme should contain at least the following template files:

- **index.php** - It serves as the *catch-all* template for WordPress pages that don't have an own template. As per convention, it should be designed to show the latest posts.
- **functions.php** - Here you can execute custom code for various WordPress actions
- **style.css** - This will either contain the style for your theme, or just a comment with information about the theme. I usually just have the comment here, as I use the styles of my homepage also for WordPress. I do this, because I have WordPress integrated into a normal html homepage.

### Optional

Further files are optional and depend on what WordPress features you use. If those files are not present and WordPress accesses a feature that requires those, it will use *index.php* instead:

- **home.php** - The main WordPress page that usually contains the list of posts (blog) or products (shop). I usually skip this file and use *index.php* to serve as home template
- **single.php** - A single blog post
- **page.php** - A generic WordPress page
- **page-[slug].php** - A custom WordPress page where *slug* should be replaced by the name of the page
- **archive.php** - Used to display category pages
- **search.php** - Used to display search results

Here you find an overview of the [WordPress template hierarchy](https://developer.wordpress.org/themes/basics/template-hierarchy/).

### Custom

You can also create custom templates, which you can select from a **Template** dropdown under **Page Attributes** whenever you create a new page - not a post.

As convention, name those templates: ``template-[name of template].php``

Here, for the name of the template use lowercase and hyphens. Example: ``template-custom.php``

At the top of a template file, you need to include the following code:

````
<?php
/**
* Template Name: name of template
*/
?>
````

Here the name of the template can also include spaces and upper case.

## Action Hooks

Action Hooks or Actions are specific points in the execution flow of WordPress where you can call custom code. It makes them an essential concept to understand and use in your WordPress themes.

Most of the time, you will use ``add_action`` to associate a custom callback function with Actions called by the WordPress core at certain times.

But there are also a few Actions which you should execute yourself inside your theme.

### Actions You Should Call

- **wp_head** - Call this Action inside the *head* section of your theme. It's mandatory for your theme to work. The **Admin Bar** will use this section, plugins might use it, and it's used to include scripts and styles. Those can be added via the [wp_enqueue_scripts](https://developer.wordpress.org/reference/hooks/wp_enqueue_scripts/) Action inside **functions.php**. Also see [wp_enqueue_script](https://developer.wordpress.org/reference/functions/wp_enqueue_script/) and [wp_enqueue_style](https://developer.wordpress.org/reference/functions/wp_enqueue_style/) to learn how to add scripts and styles inside a custom callback.
- **wp_body_open** - Call this Action after the *body* open tag of your theme.
- **wp_footer** - Call this Action inside the *footer* section of your theme.

Those Actions serve as placeholders inside your theme where WordPress will *hook* in additional code. Some of that code can be provided by your theme via calling special WordPress functions. But a lot of code will come from WordPress itself. And this code can clutter the rendered HTML pages and sometimes even introduce W3C errors.

To avoid that WordPress adds *clutter* code inside *wp_head*, use:

````
remove_action( 'wp_head','rsd_link' );
remove_action( 'wp_head','wlwmanifest_link' );
remove_action( 'wp_head','locale_stylesheet');
remove_action( 'wp_head','print_emoji_detection_script', 7);
remove_action( 'wp_head','wp_print_styles', 8);
remove_action( 'wp_print_styles', 'print_emoji_styles');
remove_action( 'wp_head','wp_print_head_scripts', 9);
remove_action( 'wp_head','wp_generator' );
remove_action( 'wp_head','rel_canonical');
remove_action( 'wp_head','wp_shortlink_wp_head', 10, 0 );
remove_action( 'wp_head','wp_custom_css_cb', 101 );
remove_filter( 'wp_robots', 'wp_robots_max_image_preview_large' ); // remove robots
function remove_api () {
    remove_action( 'wp_head', 'rest_output_link_wp_head', 10 );
    remove_action( 'wp_head', 'wp_oembed_add_discovery_links', 10 );
}
add_action( 'after_setup_theme', 'remove_api' );
````

To avoid that WordPress adds code inside *wp_footer*, use:

````
function wps_deregister_styles() {
    wp_dequeue_style( 'global-styles' );
    wp_dequeue_style( 'classic-theme-styles' );
}
add_action( 'wp_enqueue_scripts', 'wps_deregister_styles', 100 );
````

It is important to note, that you will loose functionality if you remove all WordPress hooks. So be careful and properly test your theme. 

You might also think, that not calling these hooks altogether might be a better solution than calling them and then removing the clutter in *functions.php*. But there are some lines that WordPress will insert in some pages, that are required. For example, a *Robots* setting for search result pages.

### Actions  You Hook Onto

- **after_setup_theme** - Use to perform basic setup after the theme is loaded. More information can be found [here](https://developer.wordpress.org/reference/hooks/after_setup_theme/).

### Helpful Functions

- **register_nav_menus** - Use this function to register menu *positions* which will show up in the WordPress dashboard under *Appearance -> Menus*. It works together with [wp_nav_menu](https://developer.wordpress.org/reference/functions/wp_nav_menu/) which will include a menu in the actual WordPress page where you call it inside your theme.
- **language_attributes** - Provides the language attributes for the html tag based on the *Site Language* that is set in the Wordpress admin dashboard.
- **bloginfo** - Useful to retrieve and embed information about the blog. One example would be the title. 
- **body_class** - Will attach different css classes to the body tag of a Wordpress page, based on the content. More information can be found [here](https://developer.wordpress.org/reference/functions/body_class/).

## The Main Loop

If you want to loop through the different posts in your WordPress database, you can use the following loop, which will go through a configurable number of posts and load the data. This data can then be accessed within the loop via so-called [template tags](https://codex.wordpress.org/Template_Tags).

````
if(have_posts()) :
	while(have_posts()) :
		// render your content or an excerpt of it using template tags and html
	endwhile;
else :
		// display nothing found message in HTML
endif;
````

### Template Tags

- **[the_title](https://developer.wordpress.org/reference/functions/the_title/)** - Accesses the title for the post that is currently loaded by the loop
- **[the_excerpt](https://developer.wordpress.org/reference/functions/the_excerpt/)** - Helpful in the home, category and search pages to render an excerpt of all the listed posts
- **[the_content](https://developer.wordpress.org/reference/functions/the_content/)** - Renders the complete content of the post and can also be used to render an excerpt, if the **more** flag is used in the post

