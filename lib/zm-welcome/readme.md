# Description

Easily and intuitively create WordPress admin plugin & theme landing pages.

An admin  landing page provides useful information such as;

* "getting started information"
* "whats new"
* "changelog"
* "plugin credits"

The page is normally displayed when a plugin or theme is first activated, or during an update, you can also link directly to the page for a non-intrusive user experience.

*Note the landing page is not displayed if the admin is performing a bulk update, or updating from the Network admin page.*

# Usage – Using the class

1. Require the library
2. Assign the tabs/pages
3. Assign the paths
4. Assign basic plugin info

Code example:

```
// Include the file
require_once 'lib/zm-welcome/zm-welcome.php';

function foo_plugins_loaded(){

    //
    // These are our tabs/pages, each key, i.e., about, updated, etc. represents
    // a file in our $paths array, i.e., about.php, is located in welcome/about.php
    //
    $pages = array(
        'about' => array(
            array(
                'page_title' => 'Getting Started',
            )
        ),
        'updated' => array(
            array(
                'page_title' => 'Whats New',
            )
        ),
        'changelog' => array(
            array(
                'page_title' => 'Change Log',
            )
        ),
        'credits' => array(
            array(
                'page_title' => 'Credits',
            )
        )
    );


    //
    // Define paths
    //
    $paths =  array(
        'dir_path' => plugin_dir_path( __FILE__ ) . 'welcome',
        'dir_url' => plugin_dir_url( __FILE__ ) . 'welcome'
    );

    //
    $plugin_info = array(
        'slug' => 'my-plugin',
        'version' => MY_VERSION,
        'previous_version' => get_option( MY_NAMESPACE . '_version_upgraded_from'),
        'text_domain' => MY_PLUGIN_TEXTDOMAIN,
        'start_slug' => 'getting-started'
    );

    $welcome_pages = new ZMWelcome( $pages, $paths, $plugin_info );


    //
    // This is displayed above the tabs
    //
    $welcome_pages->header  = '<div class="my-image"><img src="/welcome/assets/images/icon-256x256.png" /></div>';
    $welcome_pages->header .= '<h1>My Plugin Title</h1>';
    $welcome_pages->header .= '<div class="about-text">Intro text!</div>';


    //
    // This is the text displayed in the footer (below tabs content)
    //
    $welcome_pages->footer = '<p class="description">Have questions? Visit us.</p>';

}
add_action( 'plugins_loaded', 'foo_plugins_loaded' );
```

# Usage – Displayed with No Redirect

This approach will present the user with a link to visit the landing page once the plugin is **activated** or **update**. The link will be displayed inside of an admin notice.

*Note you will need to handle determining plugin version numbers (see below).*

```
/**
 * Show an admin notice when the plugin is activated
 * note the option *_plugin_notice_shown, is removed
 * during the 'register_deactivation_hook', see '*_deactivate()'
 */

define( 'MY_PLUGIN', 'my-plugin-slug' );
define( 'MY_PLUGIN_TEXTDOMAIN', 'my-plugin-textdomain' );

function my_plugin_admin_notice(){
    if ( ! get_option( MY_PLUGIN . '_plugin_notice_shown') && is_plugin_active( 'zm-ajax-login-register/plugin.php' ) ){

        $link = admin_url( 'welcome/welcome.php' );

        printf('<div class="updated"><p>%1$s: %2$s</p></div>',
            __('Thanks for installing be sure to check out the features',  MY_PLUGIN_TEXTDOMAIN ),
            '<a href="' . $link . '" target="_blank">Pro version</a>.'
        );
        update_option( MY_PLUGIN . '_plugin_notice_shown', 'true');
    }
}
add_action( 'admin_notices', 'my_plugin_admin_notice' );


/**
 * When the plugin is deactivated remove the shown notice option
 */
function my_plugin_register_deactivate(){
    delete_option(  MY_PLUGIN . '_plugin_notice_shown' );
}
register_deactivation_hook( __FILE__, 'my_plugin_register_deactivate' );
```

# Usage – Displayed With a Redirect

This approach will redirect the user once they activate the plugin, or once the plugin is update.

The current version number, and previous version number (upgrade from) are needed. There's various ways to obtain the version number, this is one of them.

```
code sample coming
```

# Handling Plugin Version Numbers

```
define( 'MY_NAMESPACE', 'foo' );
define( 'MY_PLUGIN_FILE', __FILE__ );
define( 'MY_VERSION', '1.0' );

/**
 * Manging of version numbers when plugin is activated
 */
function my_plugin_install() {

    // Add Upgraded From Option
    $current_version = get_option( MY_NAMESPACE . '_version' );
    if ( $current_version ) {
        update_option( MY_NAMESPACE . '_version_upgraded_from', $current_version );
    }

    update_option( MY_NAMESPACE . '_version', MY_VERSION );

    // Bail if activating from network, or bulk
    if ( is_network_admin() || isset( $_GET['activate-multi'] ) ) {
        return;
    }

    // Add the transient to redirect
    set_transient( '_' . MY_NAMESPACE . '_activation_redirect', true, 30 );
}
register_activation_hook( MY_PLUGIN_FILE, 'my_plugin_install' );


/**
 * Manging of version numbers when plugin is activated
 */
function my_plugin_deactivate() {

    delete_option( MY_NAMESPACE . '_version', MY_VERSION );

}
register_deactivation_hook( MY_PLUGIN_FILE, 'my_plugin_deactivate' );
```

# Creating the Pages

Simply create a folder in your plugin where you want to store you pages, and start creating pages! Each file will represent a tab, each file must be named the same as your tab slug, i.e. 'about' => array(), will load the about.php file.

# Styling

Simply create the following sub-directory `assets/stylesheets` in the path you've already defined in `$paths['dir_url']` with a `style.css` file in it. Example `welcome/assets/stylesheets/style.css`
