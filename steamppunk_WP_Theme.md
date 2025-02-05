## 1. Create the Theme Folder and Base Files

In your WordPress installation, navigate to `wp-content/themes` and create a folder (for example, `steampunk-newspaper`). Inside this folder, create the following required files:

- **style.css** (the theme’s main stylesheet)  
- **index.php** (a fallback template; you may leave this blank for now)  
- **functions.php** (to register theme supports, enqueue assets, and set up widget areas)  
- **header.php** (for the site’s header)  
- **footer.php** (for the site’s footer)  
- **front-page.php** (for your custom homepage layout)  

Also, create an `assets` folder with subfolders for images and JavaScript:

```
steampunk-newspaper/
├── assets/
│   ├── images/
│   │   ├── parchment.jpg        (full-screen background image)
│   │   ├── metal-texture.jpg    (optional header background texture)
│   │   └── gear.png             (gear icon for decorative/interactive use)
│   └── js/
│       └── main.js              (custom JavaScript for interactions)
├── front-page.php
├── footer.php
├── functions.php
├── header.php
├── index.php
└── style.css
```

---

## 2. Create the Theme Stylesheet (`style.css`)

At the very top of your `style.css` file, include your theme metadata. Then, add global CSS rules using CSS variables for colors so that a light/dark toggle is simple to implement. The stylesheet also includes responsive media queries for mobile devices.

**`style.css`**

```css
/*
 Theme Name: Steampunk Newspaper
 Theme URI:  http://example.com/steampunk-newspaper
 Author: Your Name
 Description: A custom Steampunk-inspired theme with an antique newspaper aesthetic, featuring a responsive three-column layout and a light/dark toggle.
 Version: 1.0
 License: GNU General Public License v3 or later
 License URI: http://www.gnu.org/licenses/gpl-3.0.html
 Text Domain: steampunk-newspaper
*/

/*---------------------------------------------
  CSS Variables for Light and Dark Modes
---------------------------------------------*/
:root {
  --background-color: #f4ecd8;      /* light parchment tone */
  --text-color: #3b2716;            /* dark brown ink */
  --header-bg: #2c2c2c;             /* dark header background */
  --header-text: #f4ecd8;           /* light header text */
  --column-bg: rgba(255,255,255,0.8);/* column overlay for readability */
  --column-border: #ccbfa1;         /* light brown border */
}

/* Default to system dark mode if applicable */
@media (prefers-color-scheme: dark) {
  :root {
    --background-color: #1e1e1e;
    --text-color: #e4e4e4;
    --header-bg: #333;
    --header-text: #e4e4e4;
    --column-bg: rgba(40,40,40,0.9);
    --column-border: #555;
  }
}

/* Allow user override via a toggled class on <body> */
body.dark-mode {
  --background-color: #1e1e1e;
  --text-color: #e4e4e4;
  --header-bg: #333;
  --header-text: #e4e4e4;
  --column-bg: rgba(40,40,40,0.9);
  --column-border: #555;
}

/*---------------------------------------------
  Global Reset and Body Styling
---------------------------------------------*/
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  background-color: var(--background-color);
  background-image: url('assets/images/parchment.jpg');
  background-size: cover;
  background-attachment: fixed;
  background-position: center;
  background-repeat: no-repeat;
  font-family: 'Libre Baskerville', serif;
  color: var(--text-color);
  line-height: 1.6;
}

/*---------------------------------------------
  Header Styles
---------------------------------------------*/
.site-header {
  padding: 20px;
  background: var(--header-bg);
  background-image: url('assets/images/metal-texture.jpg'); /* optional texture */
  background-size: cover;
  color: var(--header-text);
  position: relative;
  box-shadow: 0 0 10px rgba(0,0,0,0.5);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.site-header .branding {
  display: flex;
  align-items: center;
}

.site-header .branding img.custom-logo {
  max-height: 80px;
  margin-right: 15px;
}

.site-header .site-title {
  font-family: 'Georgia', serif;
  font-size: 2em;
  margin: 0;
}

/* Navigation Menu */
.main-menu {
  margin-top: 10px;
}

.main-menu ul {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  gap: 15px;
}

.main-menu a {
  color: var(--header-text);
  text-decoration: none;
  font-weight: bold;
}

.main-menu a:hover {
  text-decoration: underline;
}

/* Light/Dark Theme Toggle Button */
#theme-toggle {
  background: transparent;
  border: 2px solid var(--header-text);
  color: var(--header-text);
  padding: 5px 10px;
  cursor: pointer;
  font-size: 0.9em;
  border-radius: 4px;
  transition: background 0.3s, color 0.3s;
}

#theme-toggle:hover {
  background: var(--header-text);
  color: var(--header-bg);
}

/*---------------------------------------------
  Layout: Three Column Home Page
---------------------------------------------*/
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.front-columns {
  display: flex;
  gap: 20px;
  margin: 20px 0;
}

.front-columns .column {
  background: var(--column-bg);
  padding: 15px;
  border: 1px solid var(--column-border);
  box-shadow: 2px 2px 8px rgba(0,0,0,0.2);
}

.column-left, .column-right {
  width: 25%;
}

.column-center {
  width: 50%;
}

/*---------------------------------------------
  Typography & Content
---------------------------------------------*/
h1, h2, h3, h4 {
  font-family: 'Libre Baskerville', serif;
  color: var(--text-color);
  margin-top: 0.5em;
}

h1.site-title {
  font-size: 2em;
  font-weight: bold;
}

.home-heading {
  font-family: 'Courier New', Courier, monospace;
  font-weight: bold;
  font-size: 1.5em;
  margin: 1em 0 0.5em;
  border-bottom: 2px solid var(--text-color);
}

p, .post-excerpt {
  font-size: 1em;
}

.widget-title {
  font-family: 'Libre Baskerville', serif;
  font-size: 1.3em;
  margin-bottom: 0.5em;
  text-shadow: 1px 1px 0 var(--background-color);
}

/*---------------------------------------------
  Steampunk Decorative Elements
---------------------------------------------*/
/* Example drop shadow on images */
img {
  box-shadow: 0 0 10px rgba(0,0,0,0.3);
}

/*---------------------------------------------
  Responsive Media Queries
---------------------------------------------*/
@media (max-width: 768px) {
  .front-columns {
    flex-direction: column;
  }
  .front-columns .column {
    width: 100%;
    margin-bottom: 20px;
  }
  .main-menu ul {
    flex-direction: column;
    gap: 10px;
  }
  .site-header {
    flex-direction: column;
    align-items: flex-start;
  }
  #theme-toggle {
    margin-top: 10px;
  }
}
```

**Notes:**  
- We use CSS variables (with a fallback system via the `prefers-color-scheme` media query) and a `.dark-mode` class to allow toggling between light and dark themes.  
- The media query at the bottom ensures the three columns collapse into a single column on devices 768px wide or less.  
- Make sure the images (`parchment.jpg`, `metal-texture.jpg`, and `gear.png`) are placed in the `assets/images` folder.

---

## 3. Set Up `functions.php`

This file registers theme supports (logo, background, title, etc.), enqueues our CSS/JS, and registers widget areas for the three columns.

**`functions.php`**

```php
<?php
function steampunk_theme_setup() {
    // Manage document title automatically
    add_theme_support('title-tag');
    // Enable post thumbnails
    add_theme_support('post-thumbnails');
    // Enable Custom Logo support
    add_theme_support('custom-logo', array(
        'height'      => 100,
        'width'       => 400,
        'flex-height' => true,
        'flex-width'  => true,
        'header-text' => array('site-title', 'site-description'),
    ));
    // Enable Custom Background support (default parchment background)
    add_theme_support('custom-background', array(
        'default-color' => 'f4ecd8',
        'default-image' => get_template_directory_uri() . '/assets/images/parchment.jpg',
    ));
    // Enable navigation menu support
    register_nav_menu('primary', __('Primary Menu', 'steampunk-newspaper'));
}
add_action('after_setup_theme', 'steampunk_theme_setup');

function steampunk_enqueue_assets() {
    // Enqueue main stylesheet
    wp_enqueue_style('steampunk-style', get_stylesheet_uri(), array(), '1.0');
    // Enqueue Google Font (vintage serif)
    wp_enqueue_style('steampunk-fonts', 'https://fonts.googleapis.com/css2?family=Libre+Baskerville:wght@400;700&display=swap', array(), null);
    // Enqueue custom JavaScript file (placed in footer)
    wp_enqueue_script('steampunk-scripts', get_template_directory_uri() . '/assets/js/main.js', array(), '1.0', true);
}
add_action('wp_enqueue_scripts', 'steampunk_enqueue_assets');

function steampunk_register_sidebars() {
    register_sidebar(array(
        'name'          => __('Left Column', 'steampunk-newspaper'),
        'id'            => 'left-column',
        'description'   => __('Widgets in the left column on the homepage', 'steampunk-newspaper'),
        'before_widget' => '<div class="widget %2$s">',
        'after_widget'  => '</div>',
        'before_title'  => '<h3 class="widget-title">',
        'after_title'   => '</h3>',
    ));
    register_sidebar(array(
        'name'          => __('Center Column', 'steampunk-newspaper'),
        'id'            => 'center-column',
        'description'   => __('Widgets in the center column on the homepage', 'steampunk-newspaper'),
        'before_widget' => '<div class="widget %2$s">',
        'after_widget'  => '</div>',
        'before_title'  => '<h3 class="widget-title">',
        'after_title'   => '</h3>',
    ));
    register_sidebar(array(
        'name'          => __('Right Column', 'steampunk-newspaper'),
        'id'            => 'right-column',
        'description'   => __('Widgets in the right column on the homepage', 'steampunk-newspaper'),
        'before_widget' => '<div class="widget %2$s">',
        'after_widget'  => '</div>',
        'before_title'  => '<h3 class="widget-title">',
        'after_title'   => '</h3>',
    ));
}
add_action('widgets_init', 'steampunk_register_sidebars');
```

**Notes:**  
- We register support for the custom logo, background, and title.  
- A primary menu is registered so the user can set up navigation from the admin.  
- Three widget areas (for left, center, and right columns) are defined for maximum customization.

---

## 4. Build the Header Template (`header.php`)

This file outputs the HTML `<head>` with dynamic WordPress tags and renders the site header. We include a branding section, navigation menu, and now a button for toggling the light/dark theme.

**`header.php`**

```php
<!DOCTYPE html>
<html <?php language_attributes(); ?>>
<head>
  <meta charset="<?php bloginfo('charset'); ?>">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <?php wp_head(); // Loads scripts, styles, meta tags, etc. ?>
</head>
<body <?php body_class(); ?>>
<header class="site-header">
  <div class="branding">
    <?php 
      if (function_exists('the_custom_logo') && has_custom_logo()) {
        the_custom_logo(); 
      } else { 
        echo '<h1 class="site-title">' . get_bloginfo('name') . '</h1>';
      }
    ?>
  </div>
  
  <!-- Navigation Menu -->
  <?php if (has_nav_menu('primary')) : ?>
    <nav class="main-menu" role="navigation">
      <?php wp_nav_menu(array('theme_location' => 'primary')); ?>
    </nav>
  <?php endif; ?>
  
  <!-- Light/Dark Theme Toggle Button -->
  <button id="theme-toggle" aria-label="Toggle light/dark theme">Toggle Theme</button>
</header>
<div class="site-content">
```

**Notes:**  
- The `<head>` section includes the essential `wp_head()` hook.  
- The header shows a custom logo if set, or falls back to the site title.  
- The theme toggle button (with ID `theme-toggle`) will be wired up in our JavaScript to allow users to switch between light and dark modes.  
- The header and navigation are structured for responsiveness.

---

## 5. Build the Footer Template (`footer.php`)

This file wraps up your HTML and calls the required footer hook.

**`footer.php`**

```php
  </div> <!-- .site-content opened in header.php -->
  <footer class="site-footer">
    <p>&copy; <?php echo date('Y'); ?> <?php bloginfo('name'); ?>. All rights reserved.</p>
  </footer>
  <?php wp_footer(); // Loads scripts and other footer elements ?>
</body>
</html>
```

**Notes:**  
- The call to `wp_footer()` is essential for loading enqueued scripts (like our JavaScript) and any additional footer content added by plugins.

---

## 6. Create the Homepage Template (`front-page.php`)

This template file uses our three-column layout and widget areas. The center column also displays the main page content (or a loop of recent posts if you prefer).

**`front-page.php`**

```php
<?php get_header(); ?>

<div class="container front-columns">
  <!-- Left Column: Widget Area -->
  <div class="column column-left">
    <?php if (is_active_sidebar('left-column')) {
      dynamic_sidebar('left-column');
    } ?>
  </div>

  <!-- Center Column: Page Content or Latest Posts -->
  <div class="column column-center">
    <?php
      if (have_posts()) {
        the_post();
        the_content();
      }
      // Optionally, list latest posts if desired:
      if (is_home() || is_front_page()) {
        echo '<h2 class="home-heading">Latest Posts</h2>';
        $recent_posts = new WP_Query(array('posts_per_page' => 5));
        while ($recent_posts->have_posts()) : $recent_posts->the_post(); ?>
          <article id="post-<?php the_ID(); ?>" <?php post_class('home-post'); ?>>
            <h3 class="post-title"><a href="<?php the_permalink(); ?>"><?php the_title(); ?></a></h3>
            <div class="post-excerpt"><?php the_excerpt(); ?></div>
          </article>
        <?php endwhile;
        wp_reset_postdata();
      }
    ?>
  </div>

  <!-- Right Column: Widget Area -->
  <div class="column column-right">
    <?php if (is_active_sidebar('right-column')) {
      dynamic_sidebar('right-column');
    } ?>
  </div>
</div>

<?php get_footer(); ?>
```

**Notes:**  
- This template uses the widget areas for the left and right columns.  
- The center column shows the content of a static front page (or a loop of recent posts).  
- Because our CSS makes use of flexbox (with media queries for responsiveness), the columns will stack vertically on smaller screens.

---

## 7. Add Interactive JavaScript (`assets/js/main.js`)

This JavaScript file implements the light/dark toggle functionality and includes sample interactive features (like a parallax effect). It first checks for a saved user preference in `localStorage`, then falls back to the system setting via the `matchMedia` API.

**`assets/js/main.js`**

```js
document.addEventListener('DOMContentLoaded', function() {
  // Light/Dark Theme Toggle
  const toggleButton = document.getElementById('theme-toggle');
  
  // Check for saved user preference in localStorage
  const savedTheme = localStorage.getItem('theme');
  if (savedTheme) {
    if (savedTheme === 'dark') {
      document.body.classList.add('dark-mode');
    }
  } else {
    // No saved preference: default to system setting
    if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
      document.body.classList.add('dark-mode');
    }
  }
  
  if (toggleButton) {
    toggleButton.addEventListener('click', function() {
      document.body.classList.toggle('dark-mode');
      // Save the user preference
      if (document.body.classList.contains('dark-mode')) {
        localStorage.setItem('theme', 'dark');
      } else {
        localStorage.setItem('theme', 'light');
      }
    });
  }
  
  // Example: Parallax Scroll Effect for Background
  window.addEventListener('scroll', function() {
    // Adjust background position for a subtle parallax effect
    let scrollY = window.pageYOffset;
    document.body.style.backgroundPositionY = (scrollY * 0.5) + 'px';
  });
  
  // (Optional) Additional interactivity can be added here.
});
```

**Notes:**  
- When the page loads, the script checks if the user has a saved theme preference; if not, it checks the system’s `prefers-color-scheme` setting.  
- Clicking the toggle button will add or remove the `.dark-mode` class on the `<body>` element, causing the CSS variables (and thus colors) to change.  
- The parallax effect is implemented by adjusting the background position as the user scrolls.  
- This file is enqueued in `functions.php` and loaded in the footer for performance.

---

## 8. Testing and Customization

1. **Activate Your Theme:**  
   In WP Admin, go to **Appearance > Themes** and activate your “Steampunk Newspaper” theme.

2. **Customize Branding:**  
   Go to **Appearance > Customize > Site Identity** to set a custom logo. If no logo is uploaded, the site title will be displayed instead.

3. **Set Up Menus:**  
   Create a primary navigation menu in **Appearance > Menus** and assign it to the “Primary Menu” location.

4. **Widget Areas:**  
   Populate the Left, Center, and Right widget areas via **Appearance > Widgets** (or **Customize > Widgets**). These will appear in your three‑column layout.

5. **Test the Light/Dark Toggle:**  
   Use the “Toggle Theme” button in the header to switch between light and dark modes. The user’s preference is saved in localStorage. You can also test the system default by changing your OS or browser’s theme setting.

6. **Responsiveness:**  
   Resize your browser window or test on a mobile device to ensure that the columns stack gracefully and the navigation/menu adjusts as defined by the media queries.

---

## Final Thoughts

By following this tutorial you have built a complete custom WordPress theme from scratch that not only honors a unique Steampunk aesthetic with vintage textures, fonts, and decorative elements but also includes modern features such as a responsive layout and a light/dark toggle that defaults to the system setting.  
 
Remember to integrate your theme carefully with your existing WordPress installation and to keep your code version-controlled for maintainability. Continue refining your CSS and JS as needed to match your exact design vision. Happy developing!
