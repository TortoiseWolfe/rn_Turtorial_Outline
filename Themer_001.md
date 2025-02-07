Below is the complete updated Bash script. This version assumes you’re running it from the root of your WordPress installation (where the `wp-content/themes` folder exists). It creates your new theme inside `wp-content/themes/<theme-slug>`, scaffolds all standard files (including a Markdown `ReadMe.md` and a `readme.txt` for WordPress.org), creates a tailored `.gitignore`, initializes Git (and pushes if a valid remote is provided), and finally packages the theme into a ZIP file that is placed inside the theme’s own directory.

---

### Sample `.env` File

```dotenv
# .env file for WordPress theme scaffolding

# Human-readable theme name
THEME_NAME="My Awesome Theme"

# Theme slug (folder name; must be lowercase and without spaces)
THEME_SLUG="my-awesome-theme"

# Theme author
THEME_AUTHOR="TurtleWolfe@ScriptHammer.com"

# Initial theme version
THEME_VERSION="1.0.0"

# Short description of the theme
THEME_DESCRIPTION="A clean, responsive theme built for WordPress."

# Git repository URL (optional; used to initialize and push the Git repository)
REPO_URL="https://github.com/TortoiseWolfe/my-awesome-theme"
```

---

### Complete Bash Script: `create-wp-theme.sh`

```bash
#!/bin/bash
set -e

# Ensure the .env file exists
if [ ! -f .env ]; then
    echo "Error: .env file not found. Please create a .env file with the necessary variables."
    exit 1
fi

# Load environment variables from .env without splitting on spaces.
set -a
source .env
set +a

# Check required variables
REQUIRED_VARS=("THEME_NAME" "THEME_SLUG" "THEME_AUTHOR" "THEME_VERSION" "THEME_DESCRIPTION")
for var in "${REQUIRED_VARS[@]}"; do
    if [ -z "${!var}" ]; then
        echo "Error: $var is not set in .env."
        exit 1
    fi
done

# Save project root (assumed to be the root of your WordPress installation)
PROJECT_ROOT=$(pwd)

# Ensure the wp-content/themes directory exists
THEMES_DIR="wp-content/themes"
if [ ! -d "$THEMES_DIR" ]; then
    echo "Error: Directory '$THEMES_DIR' does not exist. Please run the script from the root of your WordPress installation."
    exit 1
fi

# Set the new theme directory path
THEME_DIR="$THEMES_DIR/$THEME_SLUG"
if [ -d "$THEME_DIR" ]; then
    echo "Error: Directory '$THEME_DIR' already exists."
    exit 1
fi

# Create the new theme folder inside wp-content/themes and switch to it
mkdir -p "$THEME_DIR"
cd "$THEME_DIR"

echo "Scaffolding theme structure for '$THEME_NAME' in $THEME_DIR..."

##############################
# Create style.css with header metadata
##############################
cat > style.css <<EOF
/*
 Theme Name: ${THEME_NAME}
 Theme URI: ${REPO_URL}
 Author: ${THEME_AUTHOR}
 Description: ${THEME_DESCRIPTION}
 Version: ${THEME_VERSION}
 License: GNU General Public License v2 or later
 License URI: http://www.gnu.org/licenses/gpl-2.0.html
 Text Domain: ${THEME_SLUG}
*/
EOF

echo "Created style.css"

##############################
# Create index.php
##############################
cat > index.php <<'EOF'
<?php
get_header();
?>
<div id="primary" class="content-area">
    <main id="main" class="site-main">
        <?php if ( have_posts() ) : while ( have_posts() ) : the_post(); ?>
            <article id="post-<?php the_ID(); ?>" <?php post_class(); ?>>
                <header class="entry-header">
                    <?php the_title( '<h1 class="entry-title">', '</h1>' ); ?>
                </header>
                <div class="entry-content">
                    <?php the_content(); ?>
                </div>
            </article>
        <?php endwhile; else: ?>
            <p><?php esc_html_e( 'Sorry, no posts matched your criteria.', 'textdomain' ); ?></p>
        <?php endif; ?>
    </main>
</div>
<?php
get_sidebar();
get_footer();
?>
EOF

echo "Created index.php"

##############################
# Create header.php
##############################
cat > header.php <<'EOF'
<!DOCTYPE html>
<html <?php language_attributes(); ?>>
<head>
    <meta charset="<?php bloginfo( 'charset' ); ?>">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <?php wp_head(); ?>
</head>
<body <?php body_class(); ?>>
<header id="masthead" class="site-header">
    <div class="site-branding">
        <?php
        if ( function_exists( 'the_custom_logo' ) ) {
            the_custom_logo();
        }
        ?>
        <h1 class="site-title">
            <a href="<?php echo esc_url( home_url( '/' ) ); ?>" rel="home"><?php bloginfo( 'name' ); ?></a>
        </h1>
    </div>
</header>
EOF

echo "Created header.php"

##############################
# Create footer.php
##############################
cat > footer.php <<'EOF'
<footer id="colophon" class="site-footer">
    <div class="site-info">
        <a href="<?php echo esc_url( __( 'https://wordpress.org/', 'textdomain' ) ); ?>">
            <?php printf( esc_html__( 'Proudly powered by %s', 'textdomain' ), 'WordPress' ); ?>
        </a>
    </div>
</footer>
<?php wp_footer(); ?>
</body>
</html>
EOF

echo "Created footer.php"

##############################
# Create sidebar.php
##############################
cat > sidebar.php <<'EOF'
<aside id="secondary" class="widget-area">
    <?php dynamic_sidebar( 'sidebar-1' ); ?>
</aside>
EOF

echo "Created sidebar.php"

##############################
# Create functions.php with theme support, Customizer integration, and asset enqueuing
##############################
cat > functions.php <<EOF
<?php
/**
 * ${THEME_NAME} functions and definitions.
 */

function ${THEME_SLUG}_setup() {
    // Add default posts and comments RSS feed links to head.
    add_theme_support( 'automatic-feed-links' );
    // Let WordPress manage the document title.
    add_theme_support( 'title-tag' );
    // Enable support for Post Thumbnails.
    add_theme_support( 'post-thumbnails' );
    // Enable support for custom logo.
    add_theme_support( 'custom-logo' );
    // Enable support for HTML5 markup.
    add_theme_support( 'html5', array( 'search-form', 'comment-form', 'comment-list', 'gallery', 'caption' ) );
    // Register navigation menus.
    register_nav_menus( array(
        'primary' => __( 'Primary Menu', '${THEME_SLUG}' ),
        'footer'  => __( 'Footer Menu', '${THEME_SLUG}' )
    ) );
}
add_action( 'after_setup_theme', '${THEME_SLUG}_setup' );

// Register widget area.
function ${THEME_SLUG}_widgets_init() {
    register_sidebar( array(
        'name'          => __( 'Sidebar', '${THEME_SLUG}' ),
        'id'            => 'sidebar-1',
        'description'   => __( 'Main Sidebar Area', '${THEME_SLUG}' ),
        'before_widget' => '<section id="%1$s" class="widget %2$s">',
        'after_widget'  => '</section>',
        'before_title'  => '<h2 class="widget-title">',
        'after_title'   => '</h2>',
    ) );
}
add_action( 'widgets_init', '${THEME_SLUG}_widgets_init' );

// Enqueue scripts and styles.
function ${THEME_SLUG}_scripts() {
    wp_enqueue_style( '${THEME_SLUG}-style', get_stylesheet_uri(), array(), '${THEME_VERSION}' );
    wp_enqueue_script( '${THEME_SLUG}-script', get_template_directory_uri() . '/assets/js/script.js', array(), '${THEME_VERSION}', true );
}
add_action( 'wp_enqueue_scripts', '${THEME_SLUG}_scripts' );

// Customizer additions.
function ${THEME_SLUG}_customize_register( \$wp_customize ) {
    \$wp_customize->add_section( 'sample_section', array(
        'title'    => __( 'Sample Settings', '${THEME_SLUG}' ),
        'priority' => 30,
    ) );
    \$wp_customize->add_setting( 'header_bg_color', array(
        'default'           => '#ffffff',
        'sanitize_callback' => 'sanitize_hex_color',
    ) );
    \$wp_customize->add_control( new WP_Customize_Color_Control( \$wp_customize, 'header_bg_color', array(
        'label'   => __( 'Header Background Color', '${THEME_SLUG}' ),
        'section' => 'sample_section',
    ) ) );
}
add_action( 'customize_register', '${THEME_SLUG}_customize_register' );
EOF

echo "Created functions.php"

##############################
# Create theme.json for block editor support
##############################
cat > theme.json <<EOF
{
    "version": 2,
    "settings": {
        "color": {
            "custom": true,
            "palette": []
        },
        "typography": {
            "customFontSize": true
        }
    },
    "styles": {
        "color": {},
        "typography": {}
    }
}
EOF

echo "Created theme.json"

##############################
# Create assets directory and a JavaScript file
##############################
mkdir -p assets/js
cat > assets/js/script.js <<'EOF'
// Theme JavaScript File
document.addEventListener('DOMContentLoaded', function() {
    console.log('Theme script loaded.');
});
EOF

echo "Created assets/js/script.js"

##############################
# Create a screenshot placeholder
##############################
touch screenshot.png
echo "Created screenshot.png placeholder"

##############################
# Create readme.txt (for WordPress.org repository use)
##############################
cat > readme.txt <<EOF
=== ${THEME_NAME} ===
Contributors: ${THEME_AUTHOR}
Tags: responsive, custom, theme
Requires at least: 5.0
Tested up to: 6.3
Requires PHP: 7.0
Stable tag: ${THEME_VERSION}
License: GPLv2 or later
License URI: http://www.gnu.org/licenses/gpl-2.0.html

${THEME_DESCRIPTION}

== Description ==
A brief description of the theme goes here.

== Installation ==
1. Upload the entire "${THEME_SLUG}" folder to the /wp-content/themes/ directory.
2. Activate the theme through the 'Appearance > Themes' menu in WordPress.

== Changelog ==
= ${THEME_VERSION} =
* Initial release.
EOF

echo "Created readme.txt"

##############################
# Create ReadMe.md (Markdown documentation)
##############################
cat > ReadMe.md <<EOF
# ${THEME_NAME}

**Version:** ${THEME_VERSION}  
**Author:** ${THEME_AUTHOR}  

## Description

${THEME_DESCRIPTION}

## Installation

1. Upload the entire "\${THEME_SLUG}" folder to the \`/wp-content/themes/\` directory.
2. Activate the theme via the WordPress Admin (Appearance > Themes).

## Features

- Responsive design
- Customizer integration
- Widget areas
- Navigation menu support

## Usage

Customize the theme via the WordPress Customizer. For further details, refer to the documentation.

## Changelog

- **${THEME_VERSION}**: Initial release.

## License

This theme is licensed under the [GNU General Public License v2 or later](http://www.gnu.org/licenses/gpl-2.0.html).
EOF

echo "Created ReadMe.md"

##############################
# Create license.txt with GPLv2 license text
##############################
cat > license.txt <<'EOF'
                    GNU GENERAL PUBLIC LICENSE
                       Version 2, June 1991
  Copyright (C) 1989, 1991 Free Software Foundation, Inc.
  51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA
  Everyone is permitted to copy and distribute verbatim copies
  of this license document, but changing it is not allowed.

                            Preamble

  The licenses for most software are designed to take away your
  freedom to share and change it. By contrast, the GNU General Public
  License is intended to guarantee your freedom to share and change free
  software--to make sure the software is free for all its users. This
  General Public License applies to most of the Free Software
  Foundation's software and to any other program whose authors commit to
  using it. (Some other Free Software Foundation software is covered by
  the GNU Lesser General Public License instead.) You can apply it to
  your programs, too.

  When we speak of free software, we are referring to freedom, not
  price. Our General Public Licenses are designed to make sure that you
  have the freedom to distribute copies of free software (and charge for
  this service if you wish), that you receive source code or can get it
  if you want it, that you can change the software or use pieces of it
  in new free programs; and that you know you can do these things.

  To protect your rights, we need to make restrictions that forbid
  anyone to deny you these rights or to ask you to surrender the rights.
  These restrictions translate to certain responsibilities for you if you
  distribute copies of the software, or if you modify it.

  For example, if you distribute copies of such a program, whether
  gratis or for a fee, you must pass on to the recipients the same
  freedoms that you received. You must make sure that they, too, receive
  or can get the source code. And you must show them these terms so they
  know their rights.

  Developers that use the GNU GPL protect your rights with two steps:
  (1) assert copyright on the software, and (2) offer you this License
  giving you legal permission to copy, distribute and/or modify it.

  For the developers' and authors' protection, the GPL clearly explains
  that there is no warranty for this free software. For both users' and
  authors' sake, the GPL requires that modified versions be marked as
  changed, so that their problems will not be attributed erroneously to
  authors of previous versions.

  Some devices are designed to deny users access to install or run
  modified versions of the software inside them, although the manufacturer
  can do so. This is fundamentally incompatible with the aim of
  protecting users' freedom to change the software. The systematic
  pattern of such abuse occurs in the area of products for individuals to
  use, which is precisely where it is most unacceptable. Therefore, we
  have designed this version of the GPL to prohibit the practice for those
  products. If such problems arise substantially in other domains, we
  stand ready to extend this provision to those domains in future versions
  of the GPL, as needed to protect the freedom of users.

  Finally, every program is threatened constantly by software patents.
  States should not allow patents to restrict development and use of
  software on general-purpose computers, but in those that do, we wish to
  avoid the special danger that patents applied to a free program could
  make it effectively proprietary. To prevent this, the GPL assures that
  patents cannot be used to render the program non-free.

  The precise terms and conditions for copying, distribution and
  modification follow.
  
                    GNU GENERAL PUBLIC LICENSE
   TERMS AND CONDITIONS FOR COPYING, DISTRIBUTION AND MODIFICATION

  0. This License applies to any program or other work which contains
  a notice placed by the copyright holder saying it may be distributed
  under the terms of this General Public License. The "Program", below,
  refers to any such program or work, and a "work based on the Program"
  means either the Program or any derivative work under copyright law:
  that is to say, a work containing the Program or a portion of it,
  either verbatim or with modifications and/or translated into another
  language. (Hereinafter, translation is included without limitation in
  the term "modification".) Each licensee is addressed as "you".

  1. You may copy and distribute verbatim copies of the Program's
  source code as you receive it, in any medium, provided that you
  conspicuously and appropriately publish on each copy an appropriate
  copyright notice and disclaimer of warranty; keep intact all the
  notices that refer to this License and to the absence of any warranty;
  and give any other recipients of the Program a copy of this License
  along with the Program.

  You may charge a fee for the physical act of transferring a copy, and
  you may at your option offer warranty protection in exchange for a fee.

  2. You may modify your copy or copies of the Program or any portion
  of it, thus forming a work based on the Program, and copy and
  distribute such modifications or work under the terms of Section 1
  above, provided that you also meet all of these conditions:
      a) You must cause the modified files to carry prominent notices
      stating that you changed the files and the date of any change.
      b) You must cause any work that you distribute or publish, that in
      whole or in part contains or is derived from the Program or any
      part thereof, to be licensed as a whole at no charge to all third
      parties under the terms of this License.
      c) If the modified program normally reads commands interactively
      when run, you must cause it, when started running for such
      interactive use in the most ordinary way, to print or display an
      announcement including an appropriate copyright notice and a
      notice that there is no warranty (or else, saying that you provide
      a warranty) and that users may redistribute the program under
      these conditions, and telling the user how to view a copy of this
      License. (Exception: if the Program itself is a shell script or
      script intended to be interpreted by a shell, this requirement
      does not apply to the text of the script itself.)
  
  These requirements apply to the modified work as a whole. If
  identifiable sections of that work are not derived from the Program,
  and can be reasonably considered independent and separate works in
  themselves, then this License, and its terms, do not apply to those
  sections when you distribute them as separate works. But when you
  distribute the same sections as part of a whole which is a work based
  on the Program, the distribution of the whole must be on the terms of
  this License, whose terms apply to the whole, and all its parts,
  regardless of how they are packaged. In other words, this License
  gives no permission to license the work in any other way, but it does
  not invalidate such permission if you have separately received it.
  
  In addition, mere aggregation of another work not based on the Program
  with the Program (or with a work based on the Program) on a volume of
  a storage or distribution medium does not bring the other work under
  the scope of this License.
  
  3. You may copy and distribute the Program (or a work based on it,
  under Section 2) in object code or executable form under the terms of
  Sections 1 and 2 above provided that you also do one of the following:
      a) Accompany it with the complete corresponding machine-readable
      source code, which must be distributed under the terms of this
      License, or, at your option, a written offer, valid for at least
      three years, to give anyone a complete machine-readable copy of the
      corresponding source code, to be distributed under the terms of this
      License, or
      b) Accompany it with the information you received as to the offer
      to distribute corresponding source code.
  
  4. You may not copy, modify, sublicense, or distribute the Program
  except as expressly provided under this License. Any attempt
  otherwise to copy, modify, sublicense or distribute it is void, and
  will automatically terminate your rights under this License. However,
  parties who have received copies, or rights, from you under this
  License will not have their licenses terminated so long as they
  remain in full compliance with those licenses.
  
  5. You are not required to accept this License, since you have not
  signed it. However, nothing else grants you permission to modify or
  distribute the Program or its derivative works. These actions are
  prohibited by law if you do not accept this License. Therefore, by
  modifying or distributing the Program (or any work based on the
  Program), you indicate your acceptance of this License to do so, and
  all its terms and conditions for copying, distributing or modifying
  the Program or works based on it.
  
  6. Each time you redistribute the Program (or any work based on the
  Program), the recipient automatically receives a license from the
  original licensor to copy, distribute or modify the Program subject to
  these terms and conditions. You may not impose any further
  restrictions on the recipients' exercise of the rights granted herein.
  
  7. If, as a consequence of a court judgment or allegation of patent
  infringement or for any other reason, conditions are imposed on
  distributions of the Program that contradict the conditions of this
  License, they do not excuse you from the conditions of this License.
  If you cannot distribute the Program in a manner that would be
  acceptable to the recipients under this License, then you may not
  distribute it at all.
  
  8. If the distribution of copies of the Program is made by offering
  access to copy from a designated place, then offering equivalent
  access to copy the Program from the same place is deemed to be
  distribution of copies. This License does not apply to content
  provided over a network, and you are not required to provide a copy of
  the License in such cases.
  
  9. The Free Software Foundation may publish revised and/or new versions
  of the General Public License from time to time. Such new versions will
  be similar in spirit to the present version, but may differ in detail to
  address new problems or concerns.
  
  10. Each version is given a distinguishing version number. If the
  Program specifies a version number of this License which applies to it
  and "any later version", you have the option of following the terms and
  conditions either of that version or of any later version published by
  the Free Software Foundation. If the Program does not specify a version
  number of this License, then you may choose any version ever published
  by the Free Software Foundation.
  
                 NO WARRANTY
  11. BECAUSE THE PROGRAM IS LICENSED FREE OF CHARGE, THERE IS NO WARRANTY
  FOR THE PROGRAM, TO THE EXTENT PERMITTED BY APPLICABLE LAW. EXCEPT WHEN
  OTHERWISE STATED IN WRITING THE COPYRIGHT HOLDERS AND/OR OTHER PARTIES
  PROVIDE THE PROGRAM "AS IS" WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESSED
  OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
  MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE. THE ENTIRE RISK AS
  TO THE QUALITY AND PERFORMANCE OF THE PROGRAM IS WITH YOU. SHOULD THE
  PROGRAM PROVE DEFECTIVE, YOU ASSUME THE COST OF ALL NECESSARY SERVICING,
  REPAIR OR CORRECTION.
  
                 END OF TERMS AND CONDITIONS
EOF

echo "Created license.txt"

##############################
# Create a .gitignore file tailored for a WordPress theme
##############################
cat > .gitignore <<EOF
# Ignore node modules (if using build tools)
node_modules/

# Ignore system files
.DS_Store
Thumbs.db

# Ignore build/distribution artifacts
*.zip
*.tar.gz

# Ignore environment file (if it contains sensitive configuration)
.env

# Ignore log files
*.log

# Ignore temporary directories
tmp/
EOF

echo "Created .gitignore"

##############################
# Initialize Git repository (if Git is installed)
##############################
if command -v git >/dev/null 2>&1; then
    echo "Initializing Git repository..."
    git init
    git add .
    git commit -m "Initial commit: Scaffold new WordPress theme"
    if [ -n "$REPO_URL" ]; then
        git remote add origin "$REPO_URL"
        git branch -M main
        git push -u origin main || echo "Warning: Git push failed. Check your REPO_URL and network connection."
    fi
else
    echo "Git is not installed. Skipping Git initialization."
fi

##############################
# Package the theme into a ZIP file within the same theme directory
##############################
zip -r "${THEME_SLUG}.zip" . >/dev/null
echo "Created distribution zip: ${THEME_DIR}/${THEME_SLUG}.zip"

echo "Theme scaffolding complete! Customize your theme in the '$THEME_DIR' folder."
```

---

### How to Use

1. **Create your `.env` file** in the same directory as the script (using the sample above).  
2. **Make the script executable:**
   ```bash
   chmod +x create-wp-theme.sh
   ```
3. **Run the script from the root of your WordPress installation:**
   ```bash
   ./create-wp-theme.sh
   ```

This script will now:
- Verify that the `wp-content/themes` folder exists.
- Create your theme in `wp-content/themes/<theme-slug>`.
- Scaffold all standard theme files (including a Markdown `ReadMe.md` and a `readme.txt` for WordPress.org).
- Create a tailored `.gitignore`.
- Initialize a Git repository (and push if a valid `REPO_URL` is provided).
- Package the theme into a ZIP file located in the same directory as your theme.

Happy theming, and remember to further customize the generated files as needed!
