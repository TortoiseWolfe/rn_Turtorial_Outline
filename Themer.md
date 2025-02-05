the **Bash script** that scaffolds a WordPress theme, supporting both classic PHP-based themes and block-based Full Site Editing (FSE) themes. The script will:

- **Load settings** from a `.env` file (if available) or prompt for input.
- **Initialize Git** and create a `.gitignore` file.
- **Generate necessary theme files** (`functions.php`, `style.css`, `theme.json` for block themes, and standard templates for classic themes).
- **Set up folders** for assets (`css`, `js`, `images`).
- **Enqueue styles and scripts** in `functions.php`.
- **Provide next steps** to transform the scaffolded theme into the **Steampunk Newspaper** theme.

---

## 📜 **Bash Script: `wp-theme-scaffold.sh`**
Save this script and run it in your WordPress theme directory:

```bash
#!/bin/bash

# WordPress Theme Scaffolding Script

set -e  # Exit on error

# Load .env if present, otherwise prompt for inputs
if [ -f .env ]; then
    echo "Loading environment variables from .env..."
    set -o allexport; source .env; set +o allexport
else
    echo "No .env file found. Please enter theme details manually."
    read -p "Theme Name: " THEME_NAME
    read -p "Theme Slug (folder name): " THEME_SLUG
    read -p "Theme Author: " THEME_AUTHOR
    read -p "Theme Description: " THEME_DESCRIPTION
    read -p "Use Full Site Editing (block theme)? [y/N]: " USE_FSE
fi

# Set default values if missing
THEME_SLUG=${THEME_SLUG:-$(echo "$THEME_NAME" | tr '[:upper:]' '[:lower:]' | tr ' ' '-')}
USE_FSE=${USE_FSE:-"N"}

# Create theme directory
echo "Creating theme directory: $THEME_SLUG"
mkdir -p "$THEME_SLUG"
cd "$THEME_SLUG"

# Initialize Git
echo "Initializing Git repository..."
git init

# Create .gitignore
cat <<EOL > .gitignore
node_modules/
vendor/
*.log
*.DS_Store
*.env
EOL

echo "Creating base theme files..."

# Create style.css with theme metadata
cat <<EOL > style.css
/*
Theme Name: $THEME_NAME
Theme URI: https://example.com/$THEME_SLUG/
Author: $THEME_AUTHOR
Description: $THEME_DESCRIPTION
Version: 1.0
Text Domain: $THEME_SLUG
*/
EOL

# Create assets folders
mkdir -p assets/css assets/js assets/images

# Create blank main stylesheet
touch assets/css/main.css

# Create blank JavaScript file
touch assets/js/main.js

# Create functions.php
cat <<EOL > functions.php
<?php
// Theme setup
function ${THEME_SLUG}_theme_setup() {
    add_theme_support('title-tag');
    add_theme_support('post-thumbnails');
    add_theme_support('custom-logo', array(
        'height'      => 100,
        'width'       => 400,
        'flex-height' => true,
        'flex-width'  => true,
    ));
    add_theme_support('automatic-feed-links');
}
add_action('after_setup_theme', '${THEME_SLUG}_theme_setup');

// Enqueue styles and scripts
function ${THEME_SLUG}_enqueue_assets() {
    wp_enqueue_style('${THEME_SLUG}-style', get_template_directory_uri() . '/assets/css/main.css', array(), '1.0');
    wp_enqueue_script('${THEME_SLUG}-script', get_template_directory_uri() . '/assets/js/main.js', array(), '1.0', true);
}
add_action('wp_enqueue_scripts', '${THEME_SLUG}_enqueue_assets');
EOL

# Create header.php
cat <<EOL > header.php
<!DOCTYPE html>
<html <?php language_attributes(); ?>>
<head>
    <meta charset="<?php bloginfo('charset'); ?>">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <?php wp_head(); ?>
</head>
<body <?php body_class(); ?>>
<header class="site-header">
    <div class="branding">
        <?php the_custom_logo(); ?>
        <h1 class="site-title"><?php bloginfo('name'); ?></h1>
    </div>
    <nav class="main-menu">
        <?php wp_nav_menu(array('theme_location' => 'primary')); ?>
    </nav>
</header>
<div class="site-content">
EOL

# Create footer.php
cat <<EOL > footer.php
</div> <!-- .site-content -->
<footer class="site-footer">
    <p>&copy; <?php echo date('Y'); ?> <?php bloginfo('name'); ?>. All rights reserved.</p>
</footer>
<?php wp_footer(); ?>
</body>
</html>
EOL

# Create index.php
cat <<EOL > index.php
<?php get_header(); ?>
<main>
    <h2>Welcome to $THEME_NAME</h2>
</main>
<?php get_footer(); ?>
EOL

if [[ "$USE_FSE" =~ ^[Yy]$ ]]; then
    echo "Setting up Full Site Editing (FSE) structure..."
    mkdir -p templates parts
    cat <<EOL > theme.json
{
    "version": 2,
    "settings": {
        "layout": {
            "contentSize": "840px",
            "wideSize": "1100px"
        }
    }
}
EOL
    cat <<EOL > templates/index.html
<!-- wp:template-part {"slug":"header","tagName":"header"} /-->
<!-- wp:group {"tagName":"main"} -->
    <!-- wp:post-content /-->
<!-- /wp:group -->
<!-- wp:template-part {"slug":"footer","tagName":"footer"} /-->
EOL
    cat <<EOL > parts/header.html
<!-- wp:site-title {"level":1} /-->
<!-- wp:navigation {"layout":{"type":"flex"}} /-->
EOL
    cat <<EOL > parts/footer.html
<p>© 2025 Your Site Name. All rights reserved.</p>
EOL
else
    # Classic theme: create front-page.php
    cat <<EOL > front-page.php
<?php get_header(); ?>
<div class="container">
    <h2>Welcome to $THEME_NAME</h2>
    <p>Customize this theme to your liking.</p>
</div>
<?php get_footer(); ?>
EOL
fi

echo "Theme scaffolding complete. Your theme is located in $THEME_SLUG."

```

---

## 🚀 **What To Do Next?**
Now that the scaffolding script has generated a WordPress theme, **follow these steps to turn it into the Steampunk Newspaper theme:**

1. **Edit `style.css`**  
   - Change the default colors and add a parchment background.
   - Define steampunk typography (serif fonts).
  
2. **Modify `functions.php`**  
   - Add support for **dark mode switching** and **theme customizations**.
   - Register a custom navigation menu.

3. **Customize `header.php`**  
   - Position the **logo in the top-left**.
   - Style the **navigation menu**.
   - Add a **theme toggle button** for dark/light mode.

4. **Style the theme**  
   - Open `assets/css/main.css` and:
     - Set the **background to parchment**.
     - Style a **three-column layout**.
     - Add **steampunk textures** and **drop shadows**.
  
5. **Implement JavaScript for Interactivity**  
   - Modify `assets/js/main.js` to:
     - Implement **dark mode switching**.
     - Add **hover effects** for gears.
     - Create **a subtle parallax effect**.

6. **Test and Activate**  
   - Upload the theme folder to your WordPress `wp-content/themes/` directory.
   - Activate it from the **WordPress Admin > Appearance > Themes**.

---

## 🎩 **Final Thoughts**
- This script scaffolds **both classic and block-based** themes.
- Everything is Git-tracked, so you can **commit changes** and manage versions easily.
- The next step is **styling the theme** and adding **Steampunk aesthetics**.

This should get you rolling FAST. Now, customize it and bring **Steampunk Newspaper** to life! 🚀🔥
