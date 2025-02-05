Great! I’ll create a `docker-compose.yaml` file that sets up a local WordPress development environment using a prebuilt WordPress image with MySQL. I'll ensure that your theme files are persisted while the database can be lost without issue. Additionally, I’ll provide a step-by-step tutorial on how to set up and use the environment, along with any necessary configurations for efficient WordPress theme development.

# Local WordPress Development with Docker Compose

Setting up a local WordPress environment with Docker Compose allows you to develop themes in a reproducible, isolated setup. We will use the official WordPress image (which includes Apache/PHP) and a MySQL database container. The `docker-compose.yaml` configuration will mount your theme files to ensure they persist on your host, while the database can be easily recreated if needed. Follow the steps below for a complete setup, and see the best practices section for tips on theme development in Docker.

## Docker Compose Configuration

Below is a sample **`docker-compose.yaml`** for a WordPress + MySQL development environment. It defines two services (WordPress and MySQL) and uses volumes to persist the WordPress content (themes/plugins/uploads) on your local machine:

```yaml
version: '3.8'  # Compose file version

services:
  db:
    image: mysql:5.7              # MySQL database server
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: wordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    volumes:
      - db_data:/var/lib/mysql    # (Optional) persist database data

  wordpress:
    depends_on:
      - db
    image: wordpress:latest       # WordPress + Apache + PHP
    restart: always
    ports:
      - "8000:80"                 # WordPress accessible at http://localhost:8000
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
    volumes:
      - ./:/var/www/html/wp-content  # Mount current directory as wp-content (themes/plugins/uploads)
      
volumes:
  db_data: {}   # Named volume for DB data (allows DB to persist if not removed)
```

This configuration uses the official **WordPress** image and **MySQL 5.7**. The WordPress container will automatically use Apache/PHP (the default stack in the official image) and connect to the MySQL database using the environment variables provided ([Develop your WordPress themes and plugins on Docker - DEV Community](https://dev.to/brownio/develop-your-wordpress-themes-and-plugins-on-docker-5b56#:~:text=db%3A%20image%3A%20mysql%3A5.7%20volumes%3A%20,wordpress%20MYSQL_USER%3A%20wordpress%20MYSQL_PASSWORD%3A%20wordpress))  ([Develop your WordPress themes and plugins on Docker - DEV Community](https://dev.to/brownio/develop-your-wordpress-themes-and-plugins-on-docker-5b56#:~:text=image%3A%20wordpress%3Alatest%20ports%3A%20,environment%3A%20WORDPRESS_DB_HOST%3A%20db%3A3306%20WORDPRESS_DB_USER%3A%20wordpress))  The `ports` mapping exposes WordPress on port 8000 of your localhost (feel free to change to another port if 8000 is in use). The `volumes` section is critical for development:

- **Theme & Content Mount:** The line `- ./:/var/www/html/wp-content` mounts your current project directory into the container’s WordPress content directory. This means **your local files (themes, plugins, uploads) will override and persist in the container** ([Develop your WordPress themes and plugins on Docker - DEV Community](https://dev.to/brownio/develop-your-wordpress-themes-and-plugins-on-docker-5b56#:~:text=In%20this%20case%2C%20looking%20at,content%60%20inside%20our%20Docker%20container))  Any changes you make to theme files on your host machine will appear instantly inside the container, without needing to rebuild or restart WordPress ([Guidance on using Docker with WordPress theme/plugin development - General - Docker Community Forums](https://forums.docker.com/t/guidance-on-using-docker-with-wordpress-theme-plugin-development/38042#:~:text=At%20this%20point%2C%20you%20can,dev%2Fwp_data%60%20subdirectory))  For example, if you place a custom theme in `./wp-content/themes/` on your machine, it will show up as a theme in WordPress inside the container ([Docker Wordpress plugins persistence or mapping to local plugins - Stack Overflow](https://stackoverflow.com/questions/58936160/docker-wordpress-plugins-persistence-or-mapping-to-local-plugins#:~:text=volumes%3A%20))  This ensures your theme development work is preserved on your machine, independent of the container’s lifecycle.

- **Database Storage:** The MySQL service uses a volume `db_data` at `/var/lib/mysql`. This volume will store database files on the Docker host. In a development setup, it’s okay if the database is reset or lost — you can always recreate it and re-import data if needed. The above named volume will persist the DB **unless** you remove it, so you won’t lose your database on container restarts (only if you deliberately delete the volume). This gives you flexibility: you can develop without worrying about data loss, but if something goes wrong with the DB, you can drop the volume to start fresh while your theme files remain intact.

Now that we have the compose file ready, let's go through the steps to set up and use this environment.

## Step-by-Step Setup Guide

1. **Install Docker and Docker Compose:** Ensure you have Docker Desktop or Docker Engine installed on your system. Docker Compose is included with Docker Desktop (Compose V2) on modern installations. You can verify by running `docker --version` and `docker compose version` in your terminal ([Develop your WordPress themes and plugins on Docker - DEV Community](https://dev.to/brownio/develop-your-wordpress-themes-and-plugins-on-docker-5b56#:~:text=%2A%20Clone%20the%20wp,WP%20is%20up%20%26%20running))  If not installed, download Docker from the official site and follow the installation instructions for your OS.

2. **Create a Project Directory:** Choose a location on your machine for your WordPress development project. For example, you might create a directory called `wp-theme-dev`:
   ```bash
   mkdir wp-theme-dev && cd wp-theme-dev
   ```
   This directory will hold your WordPress content (themes/plugins) and the `docker-compose.yaml` file.

3. **Add WordPress Content (Optional):** If you already have a WordPress theme (or a whole `wp-content` folder) you want to work on, copy it into this project directory. For instance, ensure your theme resides in `wp-theme-dev/wp-content/themes/your-theme`. If you don't have any files yet, Docker will create a fresh `wp-content` structure in the mounted volume when the containers run ([Docker Wordpress plugins persistence or mapping to local plugins - Stack Overflow](https://stackoverflow.com/questions/58936160/docker-wordpress-plugins-persistence-or-mapping-to-local-plugins#:~:text=volumes%3A%20))  Either way, the current directory will serve as the `wp-content` inside the container.

4. **Create the `docker-compose.yaml`:** In the `wp-theme-dev` directory, create a file named `docker-compose.yaml` (or `docker-compose.yml`) and paste the configuration from the previous section. Save the file. This defines the WordPress and MySQL services, as discussed:
   - WordPress service depends on the MySQL service and will use the credentials we provided.
   - MySQL service will initialize a database named "wordpress" with the username/password "wordpress" (as set in the environment).  

   Make sure the indentation and syntax are correct in the YAML file. (Refer to the example above or sources if needed ([Develop your WordPress themes and plugins on Docker - DEV Community](https://dev.to/brownio/develop-your-wordpress-themes-and-plugins-on-docker-5b56#:~:text=db%3A%20image%3A%20mysql%3A5.7%20volumes%3A%20,wordpress%20MYSQL_USER%3A%20wordpress%20MYSQL_PASSWORD%3A%20wordpress))  ([Develop your WordPress themes and plugins on Docker - DEV Community](https://dev.to/brownio/develop-your-wordpress-themes-and-plugins-on-docker-5b56#:~:text=image%3A%20wordpress%3Alatest%20ports%3A%20,environment%3A%20WORDPRESS_DB_HOST%3A%20db%3A3306%20WORDPRESS_DB_USER%3A%20wordpress)) )

5. **Launch the Docker Environment:** In your terminal, from the `wp-theme-dev` directory, run:
   ```bash
   docker compose up -d
   ``` 
   This command starts Docker Compose in detached mode, downloading the WordPress and MySQL images if you don't have them already. The first run may take a few minutes as images are pulled and containers are created ([Develop your WordPress themes and plugins on Docker - DEV Community](https://dev.to/brownio/develop-your-wordpress-themes-and-plugins-on-docker-5b56#:~:text=Well%2C%20this%20is%20pretty%20straightforward,will%20be%20created%20and%20started))  Once done, you should have two containers running: one for the database and one for WordPress. (You can check with `docker compose ps` to see the running services.)

6. **Access WordPress Setup:** Open your web browser and navigate to **http://localhost:8000** (or the port you configured). You should see the WordPress setup page (language selection or installation screen) load. Complete the WordPress installation by entering a site title, admin username, password, and email. (Since this is a dev environment, you can use something simple like *admin/admin* for ease, but remember these credentials.) Once you finish the install, you will arrive at the WordPress dashboard ([Develop your WordPress themes and plugins on Docker - DEV Community](https://dev.to/brownio/develop-your-wordpress-themes-and-plugins-on-docker-5b56#:~:text=This%20means%20everything%20is%20good,work%20on%20anything%20you%20want)) 

7. **Develop Your Theme:** Now you have a running WordPress site. In the dashboard, go to **Appearance > Themes** and you should see your theme listed (if you placed it in the `wp-content/themes` directory earlier). Activate the theme. You can now begin developing the theme:
   - Open the theme files in your code editor from your host machine (e.g., the `wp-theme-dev/wp-content/themes/your-theme` folder).
   - Edit template files, stylesheets, or PHP code as needed. Because of the volume mount, **every change is immediately reflected inside the container**. For example, editing `functions.php` or CSS in your editor will update the file in the Docker container in real time ([Develop your WordPress themes and plugins on Docker - DEV Community](https://dev.to/brownio/develop-your-wordpress-themes-and-plugins-on-docker-5b56#:~:text=In%20this%20case%2C%20looking%20at,content%60%20inside%20our%20Docker%20container))  You can simply refresh the browser to see your changes – no need to rebuild the container.
   - If you need to add plugins for testing, you can install them via the WP Admin or by placing plugin files in `wp-content/plugins` (which is also within the mounted volume). These too will persist on your host.

8. **Stopping and Starting the Environment:** When you are done with development for the day, you can stop the containers with:
   ```bash
   docker compose down
   ```
   This will stop and remove the containers (but not the volumes). Your theme files remain on your disk, and the database data remains in the `db_data` volume. To start the environment again later, simply run `docker compose up -d` again, and WordPress will be running with the same data (it will reuse the existing database volume if present). You won’t need to reinstall WordPress each time since the database was persisted — unless you intentionally removed the volume to reset the database.

## Best Practices for Theme Development in Docker

Developing a WordPress theme in a Docker-based environment is convenient and portable. Here are some best practices and tips to get the most out of this setup:

- **Persist Only What Matters:** We mounted the entire `wp-content` directory to preserve themes (and plugins/uploads). This means your code changes are safe on the host. The database, on the other hand, can be treated as disposable in development. If something goes wrong with your WordPress installation or you want a clean slate, you can remove the `db_data` volume and re-run the setup without losing your theme work. In other words, **keep persistent files (code and media) on your host, but don’t worry about wiping the database for a fresh install** when needed ([Guidance on using Docker with WordPress theme/plugin development - General - Docker Community Forums](https://forums.docker.com/t/guidance-on-using-docker-with-wordpress-theme-plugin-development/38042#:~:text=At%20this%20point%2C%20you%20can,dev%2Fwp_data%60%20subdirectory)) 

- **Use Version Control for Your Theme:** Keep your theme in a version control system (e.g., Git). Since the theme files live on your host machine (and are merely shared with the container), you can easily initialize a git repository in your theme folder. This helps collaborate with team members and track changes, independent of the Docker environment.

- **Leverage WP Debugging:** For development, it’s recommended to enable WordPress debug mode. You can do this by adding an environment variable in your compose file under the WordPress service:
  ```yaml
  WORDPRESS_DEBUG: 1
  ``` 
  This sets `WP_DEBUG` to true inside WordPress (the official image translates that env var into the wp-config configuration) ([Docker Wordpress plugins persistence or mapping to local plugins - Stack Overflow](https://stackoverflow.com/questions/58936160/docker-wordpress-plugins-persistence-or-mapping-to-local-plugins#:~:text=WORDPRESS_AUTH_KEY%3A%205f6ede1b94d25a2294e29eeba929a8c80a5ac0fb%20WORDPRESS_SECURE_KEY%3A%205f6ede1b94d25a2294e29eeba929a8c80a5ac0fb%20WORDPRESS_LOGGED_IN_KEY%3A,5f6ede1b94d25a2294e29eeba929a8c80a5ac0fb%20WORDPRESS_NONCE_SALT%3A%205f6ede1b94d25a2294e29eeba929a8c80a5ac0fb%20WORDPRESS_DEBUG%3A%201))  With debug mode on, WordPress will log errors and warnings, helping you catch issues in your theme code quickly. (You might also want to set `WP_DEBUG_LOG` or other debug constants if needed – these can be added similarly as env variables.)

- **Optional Tools (Advanced):** For easier database inspection, you can add a phpMyAdmin service to your compose file (using the `phpmyadmin/phpmyadmin` image) configured to connect to the `db` container. This isn't required, but it can be handy to peek into your MySQL database via a web interface. Another useful tool is **WP-CLI** – the official WP command-line tool. You can run WP-CLI commands by using `docker compose run --rm wordpress wp <command>` or by exec-ing into the WordPress container, since the official image comes with WP-CLI installed. These tools can streamline tasks like installing plugins, creating users, or resetting the database from the command line.

- **Sync Across Team and Environments:** Because everything is defined in `docker-compose.yaml`, all developers on your team can use the same setup. Commit the compose file (and perhaps a stub `wp-content` structure or a README) to your project repository. Anyone can then clone the repo and run `docker compose up` to replicate the environment. The uniform environment avoids the "works on my machine" problems and ensures everyone is developing against the same stack ([Guidance on using Docker with WordPress theme/plugin development - General - Docker Community Forums](https://forums.docker.com/t/guidance-on-using-docker-with-wordpress-theme-plugin-development/38042#:~:text=1,the%20files%20in%20this%20subdirectory))  Just share theme/plugin code via version control, and each developer can run the containers locally.

- **Avoid Production Use:** This Docker setup is meant for **development only**. Do not use the default credentials and configuration in production. For instance, using a root password of "wordpress" and exposing ports without security is fine on localhost but not safe for a live server. When it’s time to deploy, you would handle WordPress and database with proper configurations outside of this simple dev compose file. *Treat this environment as disposable and for convenience.* It lets you focus on theme development without worrying about installing PHP/Apache/MySQL on your machine ([Docker environment for WordPress development - DEV Community](https://dev.to/sarahcssiqueira/docker-environment-for-wordpress-development-p46#:~:text=Is%20meant%20to%20be%20used,to%20be%20used%20in%20production)) 

By following this guide, you can rapidly spin up a WordPress site for theme development and tear it down when finished, all while keeping your theme files intact on your local drive. The Docker-based approach ensures a consistent environment (Apache, PHP, MySQL versions, etc.) for testing your theme. You can develop confidently knowing that if the database is corrupted or you want to start over, it's just a matter of resetting the DB container or volume, with no impact on your theme code. Happy developing!

**Sources:**

- Docker Compose example for WordPress (WordPress + MySQL) ([Develop your WordPress themes and plugins on Docker - DEV Community](https://dev.to/brownio/develop-your-wordpress-themes-and-plugins-on-docker-5b56#:~:text=db%3A%20image%3A%20mysql%3A5.7%20volumes%3A%20,wordpress%20MYSQL_USER%3A%20wordpress%20MYSQL_PASSWORD%3A%20wordpress))  ([Develop your WordPress themes and plugins on Docker - DEV Community](https://dev.to/brownio/develop-your-wordpress-themes-and-plugins-on-docker-5b56#:~:text=image%3A%20wordpress%3Alatest%20ports%3A%20,environment%3A%20WORDPRESS_DB_HOST%3A%20db%3A3306%20WORDPRESS_DB_USER%3A%20wordpress))  
- Explanation of mounting host `wp-content` into the container for live development ([Develop your WordPress themes and plugins on Docker - DEV Community](https://dev.to/brownio/develop-your-wordpress-themes-and-plugins-on-docker-5b56#:~:text=In%20this%20case%2C%20looking%20at,content%60%20inside%20our%20Docker%20container))  ([Guidance on using Docker with WordPress theme/plugin development - General - Docker Community Forums](https://forums.docker.com/t/guidance-on-using-docker-with-wordpress-theme-plugin-development/38042#:~:text=At%20this%20point%2C%20you%20can,dev%2Fwp_data%60%20subdirectory))  
- Docker forum – using Docker for WordPress theme development (volume mounts and workflow) ([Guidance on using Docker with WordPress theme/plugin development - General - Docker Community Forums](https://forums.docker.com/t/guidance-on-using-docker-with-wordpress-theme-plugin-development/38042#:~:text=1,the%20files%20in%20this%20subdirectory))  ([Guidance on using Docker with WordPress theme/plugin development - General - Docker Community Forums](https://forums.docker.com/t/guidance-on-using-docker-with-wordpress-theme-plugin-development/38042#:~:text=At%20this%20point%2C%20you%20can,dev%2Fwp_data%60%20subdirectory))  
- Stack Overflow – persisting WordPress theme/plugins with Docker volumes ([Docker Wordpress plugins persistence or mapping to local plugins - Stack Overflow](https://stackoverflow.com/questions/58936160/docker-wordpress-plugins-persistence-or-mapping-to-local-plugins#:~:text=volumes%3A%20))  
- Dev.to tutorial – “Develop your WordPress themes and plugins on Docker” ([Develop your WordPress themes and plugins on Docker - DEV Community](https://dev.to/brownio/develop-your-wordpress-themes-and-plugins-on-docker-5b56#:~:text=In%20this%20case%2C%20looking%20at,content%60%20inside%20our%20Docker%20container))  ([Develop your WordPress themes and plugins on Docker - DEV Community](https://dev.to/brownio/develop-your-wordpress-themes-and-plugins-on-docker-5b56#:~:text=Well%2C%20this%20is%20pretty%20straightforward,will%20be%20created%20and%20started))  
- Stack Overflow – enabling WP_DEBUG mode via docker-compose env ([Docker Wordpress plugins persistence or mapping to local plugins - Stack Overflow](https://stackoverflow.com/questions/58936160/docker-wordpress-plugins-persistence-or-mapping-to-local-plugins#:~:text=WORDPRESS_AUTH_KEY%3A%205f6ede1b94d25a2294e29eeba929a8c80a5ac0fb%20WORDPRESS_SECURE_KEY%3A%205f6ede1b94d25a2294e29eeba929a8c80a5ac0fb%20WORDPRESS_LOGGED_IN_KEY%3A,5f6ede1b94d25a2294e29eeba929a8c80a5ac0fb%20WORDPRESS_NONCE_SALT%3A%205f6ede1b94d25a2294e29eeba929a8c80a5ac0fb%20WORDPRESS_DEBUG%3A%201))  

