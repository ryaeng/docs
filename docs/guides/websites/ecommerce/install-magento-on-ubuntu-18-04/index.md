---
slug: install-magento-on-ubuntu-18-04
description: "Install Magento Community Edition on Ubuntu 18.04 to manage your e-commerce site."
keywords: ["magento", "ubuntu", "e-commerce", "magento ubuntu"]
tags: ["ubuntu", "ssl", "lamp", "cms"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
modified: 2018-08-29
modified_by:
  name: Andrew Lescher
published: 2016-12-28T00:00:00.000Z
title: Install Magento on Ubuntu 18.04
external_resources:
  - '[Magento Documentation](http://docs.magento.com/m2/ce/user_guide/getting-started.html)'
  - '[Magento Resources Library](https://magento.com/resources)'
relations:
    platform:
        key: how-to-install-magento
        keywords:
           - distribution: Ubuntu 18.04
aliases: ['/websites/ecommerce/install-magento-on-ubuntu-18-04/']
authors: ["Linode"]
---

In this guide you'll learn how to install Magento on Ubuntu 18.04\. Magento Community Edition (CE) is a free, open-source e-commerce platform. It's one of the most popular solutions for self-hosted online stores due to its simple yet powerful admin panel and large developer community.

Considering the resources some Magento plugins demand, we strongly recommend that you have at least a **Linode 4GB**. You'll need to allocate up to 2GB of memory for PHP and Magento; running Magento on a smaller Linode may result in server crashes or unreliability under medium to heavy traffic. For more memory-intensive Magento setups, we recommend using a [High Memory Linode](https://www.linode.com/pricing/).

{{< note respectIndent=false >}} This guide explains how to install the latest Magento release at the time of publication. For the Community Edition, this will be version 2.2.x. If you plan to use data, themes, and extensions from an older Magento site, be sure to check for compatibility issues between the two versions. {{< /note >}}

## Before You Begin

1.  Familiarize yourself with our [Getting Started](/docs/products/platform/get-started/) guide and complete the steps for setting your Linode's hostname and timezone.

2.  Complete the sections of our [Securing Your Server](/docs/products/compute/compute-instances/guides/set-up-and-secure/) guide to create a standard user account, harden SSH access, and remove unnecessary network services.

3.  Magento runs on a LAMP stack, and is **only compatible with PHP 7.0 or 7.1**. This guide contains instructions for setting up Apache, MySQL, and PHP in a manner compatible with the latest version of Magento (v. 2.2.5). Check your version of PHP with `php -v`. If your version is incompatible, refer to the included instructions for installing and setting PHP 7.0 as the default version.

    {{< note respectIndent=false >}} If you have an existing LAMP stack installed on your Linode, refer to Magento's [prerequisites](https://devdocs.magento.com/guides/v2.2/install-gde/prereq/prereq-overview.html) to ensure a particular version's compatibility with the components of your stack. {{< /note >}}

4.  Update your system:

    ```
    sudo apt update && sudo apt upgrade
    ```

    {{< note respectIndent=false >}} This guide is written for a non-root user. Commands that require elevated privileges are prefixed with `sudo`. If you're not familiar with the `sudo` command, you can check our [Users and Groups](/docs/guides/linux-users-and-groups/) guide. {{< /note >}}

## Install A Magento-compatible LAMP Stack

1.  Install Apache 2.4.

    ```
    apt install apache2
    systemctl enable apache2
    ```

2.  Install MariaDB 10.2

    {{< note respectIndent=false >}} Magento is also compatible with MySQL 5.7.x, MySQL NDB Cluster 7.4.x, MariaDB 10.0, 10.1, 10.2, Percona 5.7 and other binary compatible MySQL technologies. You may choose to use any one of the listed versions.  {{< /note >}}

    ```
    sudo apt-get install software-properties-common
    sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
    sudo add-apt-repository 'deb [arch=amd64] http://mariadb.biz.net.id/repo/10.2/ubuntu bionic main'
    ```

    Update packages and install MariaDB.

    ```
    apt update
    apt install mariadb-server
    ```

    Setup MariaDB using the mysql_secure_installation built-in shell script. Supply each prompt with `y` and hit `Enter`. Set a secure password for the *root* user when prompted.

    ```
    mysql_secure_installation
    ```

    Set MariaDB to start on system boot and initialize the daemon.

    ```
    systemctl enable mysql
    systemctl start mysql
    ```

3.  Install the PHP 7.0 repository and update packages.

    ```
    sudo add-apt-repository ppa:ondrej/php
    sudo apt update -y
    ```

    Install all required PHP extensions.

    ```
    sudo apt install -y php7.0 libapache2-mod-php7.0 php7.0-common php7.0-gd php7.0-mysql php7.0-mcrypt php7.0-curl php7.0-intl php7.0-xsl php7.0-mbstring php7.0-zip php7.0-bcmath php7.0-iconv php7.0-soap
    ```

4.  **Optional** - Proceed with this step only if you already have an incompatible PHP version installed and need to switch.

    Replace `php7.x` with your incompatible PHP version in the first command.

    ```
    sudo a2dismod php7.x && sudo a2enmod php7.0
    sudo systemctl restart apache2
    sudo update-alternatives --set php /usr/bin/php7.0
    ```

    Confirm your version of PHP with `php -v` and verify that the output matches the following:

    ```
    PHP 7.0.31-1+ubuntu18.04.1+deb.sury.org+1 (cli) (built: Jul 25 2018 10:01:10) ( NTS )
    Copyright (c) 1997-2017 The PHP Group
    Zend Engine v3.0.0, Copyright (c) 1998-2017 Zend Technologies
        with Zend OPcache v7.0.31-1+ubuntu18.04.1+deb.sury.org+1, Copyright (c) 1999-2017, by Zend Technologies
    ```

## Configure Apache

Since Magento will be served by Apache, some additional configuration is needed to ensure the application is served properly. In this section, we'll configure Apache for Magento to ensure that styles and other settings display properly in your storefront.

1.  Confirm you're using Apache version 2.4:

    ```
    apache2 -v
    ```

    If this shows version 2.2 or another version, upgrade Apache before proceeding.

2.  Enable the Apache rewrite module:

    ```
    sudo a2enmod rewrite
    ```

3.  Modify the virtual host file for your Magento site to resemble the example below. If you have not previously created a virtual host file, do so now and refer to the [Configure Virtual Hosts](/docs/guides/install-lamp-stack-on-ubuntu-16-04/#configure-virtual-hosts) section of the LAMP on Ubuntu 16.04 guide for additional guidance.

    {{< file "/etc/apache2/sites-available/example.com.conf" apache >}}

<directory /var/www/html/example.com/public_html>
    Require all granted
</directory>

<virtualhost *:80>
    ServerName example.com
    ServerAlias www.example.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/example.com/public_html

    ErrorLog /var/www/html/example.com/logs/error.log
    CustomLog /var/www/html/example.com/logs/access.log combined

    <directory /var/www/html/example.com/public_html>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
    </directory>
</virtualhost>

{{< /file >}}

    The `Directory` block inside the `Virtual Host` block should point to the directory where you plan to install Magento. For simplicity, we will be installing it in our web root, but if you want to put it elsewhere (e.g., a subdirectory of your web root), modify this setting.

    It's important to note the value of `AllowOverride` as this affects which settings in each directory's `.htaccess` file will be applied and which will be ignored. If you're not sure if the `All` option is optimal for your site, refer to the [Apache documentation](http://httpd.apache.org/docs/current/mod/core.html#allowoverride) for more information on this setting.

4.  If you haven't already, enable your site:

    ```
    sudo a2ensite example.com
    ```

    Replace `example.com` with the name of your site's virtual host.

5.  Restart Apache to apply these changes:

    ```
    sudo systemctl restart apache2
    ```

6.  **Optional**: You may want to take this opportunity to create a `phpinfo.php` page to ensure that PHP is active and working properly with Apache:

    {{< file "/var/www/html/example.com/public_html/phpinfo.php" php >}}<?php phpinfo(); ?>{{< /file >}}

    Once you've visited this page in your browser and confirmed that PHP is working, delete the test page.

## Create a MySQL Database and User

If you previously installed a LAMP stack using this or our other guides you should already have set up MySQL root login credentials. If not, you'll need to go back and do so before proceeding.

1.  Log into the MySQL shell as the root user, entering your password when prompted:

    ```
    mysql -u root -p
    ```

2.  Create a Magento database and user, and set the permissions. In this example, we'll call our database and user `magento`. Replace `password` with a secure password. You may optionally replace the other values as well:

    ```
    CREATE DATABASE magento;
    CREATE USER 'magento' IDENTIFIED BY 'password';
    GRANT ALL PRIVILEGES ON magento.* TO 'magento';
    ```

    This section assumes that your database is hosted on the same server as your Magento application. If this is not the case, perform these steps and then refer to Magento's guide on using a [remote database server](https://devdocs.magento.com/guides/v2.2/install-gde/prereq/mysql_remote.html).

3.  Exit the MySQL shell:

    ```
    quit
    ```

## Install Magento

### Download Magento Community Edition

In this section, we'll explain how to install the Magento Community Edition (CE) software on your Linode. There are several ways to do this, but here we'll cover the simplest method. Before starting this section, you must create an account to gain access to the Magento software. Create an account [here](https://account.magento.com/customer/account/create/).

1.  On your local computer, download a copy of Magento from their [official website](https://www.magentocommerce.com/download). Several versions are available, but we recommend downloading the latest release. As of this writing, the latest version is 2.2.5.

    ![The Magento download screen](magento-download-screen.png)

    Next to your selected version, there will be a dropdown menu that says "Select your format." Select the option ending with the `.tar.gz` extension and click the **Download** button. Note that in this step, you will be presented with the option to download the software with optional sample data. Whether you choose this or the basic version is up to you.

    ![Select the option ending in ".tar.gz"](magento-tar-gz.png)

2.  From your local computer, copy the file to your Linode. Replace */path/on/local/* with the path of your downloaded file, *user* with your standard user account on your Linode, and *host-ip_or_web-url* with your Linode’s web-url or IP address:

    ```
    scp /path/on/local/Magento-CE-2.*.tar.gz user@host-ip_or_web-url:~/
    ```

    Alternatively, you can use an FTP client, like [Filezilla](/docs/guides/filezilla/), if you're running Windows or are otherwise unable to use a command-line tool like `scp`.

3.  Log into your Linode via SSH as your standard user account. Navigate to the document root you specified in your virtual host file:

    ```
    cd /var/www/html/example.com/public_html
    ```

    Extracting the archive directly into your webroot, as you'll do in the next step, will make it accessible as the main page on your domain. For instance, if your domain is `example.com`, the Magento storefront will display when you visit `example.com` in your browser.

    If you want to run a Magento store as a subsection of your site, move and extract the archive in a directory within your webroot. For instance, if you intend to make your site accessible by visiting `example.com/store`, create the subdirectory `/var/www/html/example.com/public_html/store` and navigate to that directory before proceeding to the next step.

4.  Move the Magento archive into your document root and extract its contents:

    ```
    sudo mv ~/Magento-CE-2.*.tar.gz .
    sudo tar -xvf Magento-CE-2.*.tar.gz
    ```

## Configure Users and Permissions

1.  Create a Magento user, which will run the software. For simplicity, call this user `magento`:

    ```
    sudo useradd magento
    ```

2.  Next, add the Magento user to the web server's user group. For Apache, the default user is `www-data`:

    ```
    sudo usermod -g www-data magento
    ```

3.  The commands in this step should be run from your Magento installation directory (where you extracted the archive). If you are not still in that directory, navigate there before proceeding.

    Run these commands in order:

    ```
    sudo find var vendor pub/static pub/media app/etc -type f -exec chmod g+w {} \;
    sudo find var vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} \;
    sudo chown -R magento:www-data .
    sudo chmod u+x bin/magento
    ```

    This allows your `magento` user (and members of the `www-data` group) to write to the various files they need to run and serve Magento on your site.

    {{< note respectIndent=false >}} The first two commands may take some time to run because they are matching various files and directories in your installation folder. It may appear that the system is inactive, but be sure to allow a couple of minutes before cancelling the operations. You can always start a new SSH connection, or run these commands in a [screen](/docs/guides/using-gnu-screen-to-manage-persistent-terminal-sessions/) session, and use tools like [htop & iotop](/docs/guides/top-htop-iotop/#additional-top-like-programs) to monitor progress. {{< /note >}}

4.  Restart Apache:

    ```
    sudo systemctl restart apache2
    ```

## Install Magento CE

1.  Switch to the `magento` user and navigate to the `bin` directory in your Magento installation folder:

    ```
    sudo su magento
    cd bin
    ```

2.  Run the Magento installation script with the following options:


    ```
    ./magento setup:install --admin-firstname="John" --admin-lastname="Doe" --admin-email="your@email.com" --admin-user="john" --admin-password="password1" --db-name="magento" --db-host="localhost" --db-user="magento" --db-password="password"
    ```

    Replace the values in the options as follows:

    - **admin-firstname** / **admin-lastname** - This will set the full name of your admin user. Replace these with your name if you'll be the administrator.
    - **admin-email** - This is your email for resetting passwords and receiving admin notifications.
    - **admin-user** / **admin-password** - These are your login credentials for the Magento admin control panel. Be sure to set a secure password here and store it in a safe place. Note that the Magento script requires you to use both letter and number characters in your password, and will return an error if you do not.
    - **db-name** - This is the name of the database you set up in MySQL. In our example, we called it `magento`, but if you chose a different value, substitute it here.
    - **db-host** - If you're running Magento on the same server as its database, use `localhost` here. If not, this value will be the hostname of the server on which your database lives.
    - **db-user** - This is the MySQL database user you set up previously. In our example, we called it `magento` but if you chose a different name, use that here instead.
    - **db-password** - This will be the password you configured for the `magento` MySQL user.

    {{< note respectIndent=false >}} These are just a few of the available options to configure your Magento installation. For more information, refer to the [Magento Installation Guide](http://devdocs.magento.com/guides/v2.1/install-gde/install/cli/install-cli-install.html) and feel free to use additional options when running the script. {{< /note >}}

3.  The installation script may take several minutes to run. Once it's finished, you'll see a success message:

    ```
    [SUCCESS]: Magento installation complete. [SUCCESS]: Magento Admin URI: /admin_a61e40
    ```

   You've successfully installed Magento on your Linode. You can log into your admin panel by entering your domain, followed by the "Magento Admin URI" displayed above, in a web browser. The **admin-user** and **admin-password** options you specified when running the installation script will be your credentials.

4.  Exit from the `magento` user:

    ```
    exit
    ```

## Configure Magento

The dashboard is functional at this point, but you've still got work to do before the site is ready to use. In this section, we'll explain how to set up cron jobs and secure the Magento software to be suitable for a live e-commerce site.

### Set Cron Jobs

Magento relies on [cron](/docs/guides/schedule-tasks-with-cron/) to perform tasks like continuously reindexing your site and generating emails and newsletters. If you logged into your admin panel, you may have noticed an error message saying that cron jobs needed to be set. Fortunately, the cron jobs Magento uses for a base installation are very easy to configure.

1.  Open the crontab for your `magento` user. Perform this step as a user with `sudo` privileges:

    ```
    sudo crontab -u magento -e
    ```

2.  If this is your first time using cron, you'll be prompted to select a text editor. If you don't have a preference, select **2** to use nano.

3.  Add the following lines to the end of the file, substituting your Magento installation directory in each:

        * * * * * /usr/bin/php /var/www/html/example.com/public_html/bin/magento cron:run | grep -v "Ran jobs by schedule" >> /var/www/html/example.com/public_html/var/log/magento.cron.log

        * * * * * /usr/bin/php /var/www/html/example.com/public_html/update/cron.php >> /var/www/html/example.com/public_html/var/log/update.cron.log

        * * * * * /usr/bin/php /var/www/html/example.com/public_html/bin/magento setup:cron:run >> /var/www/html/example.com/public_html/var/log/setup.cron.log

    The first line handles reindexing. The second and third lines are needed by the Component Manager and System Upgrade, respectively. The `/usr/bin/php` part of each line specifies your PHP binary. If you installed PHP using our LAMP stack guide, you can leave this value as we have shown it, but if you installed PHP by some other method, you can check its location with `which php` and substitute that value.

    When you're done, save and exit the file.

To verify that the rules have been set properly, log out of your Magento admin and log back in. If everything has been configured correctly, you should no longer see the notification.

For more information about setting up cron jobs for development servers and custom Magento modules, refer to the [Magento Cron Documentation](http://devdocs.magento.com/guides/v2.1/config-guide/cli/config-cli-subcommands-cron.html#config-cli-cron-bkg).

### Configure X-Frame Options

We strongly recommend disabling the ability to display your Magento storefront in a frame to prevent [clickjacking](https://en.wikipedia.org/wiki/Clickjacking) attacks. To do this, modify the following line in your `env.php` file:

{{< file "/var/www/html/example.com/public_html/app/etc/env.php" php >}} 'x-frame-options' => 'DENY',

{{< /file >}}

This prevents attackers from embedding your site in a frame (for example, on a malicious site that mimics your store) in an attempt to intercept payment and other sensitive customer information.

### Set Directory Permissions

At a minimum, you should restrict write access to the `app/etc` directory before opening the site to customers. Run these commands from within your Magento installation directory:

```
sudo find app/etc -type f -exec chmod g-w {} \;
sudo find app/etc -type d -exec chmod g-ws {} \;
```

Depending on whether you install custom themes or extensions, you may need to do additional configuration. This will vary depending on what you have installed. Once you're ready to deploy your site into production mode, refer to [Magento's ownership and permissions guide](https://devdocs.magento.com/guides/v2.2/config-guide/prod/prod_file-sys-perms.html) for a more comprehensive set of recommendations.

{{< note respectIndent=false >}} If you need to make additional configuration changes in the future, you'll need to manually add write permissions again before you can do so. For more information, see our guide on [Linux users and groups](/docs/guides/linux-users-and-groups/). {{< /note >}}

### Secure your Site with SSL

Secure sockets layer (SSL) certificates are a vital part of e-commerce. They enable encrypted transmission of sensitive data, such as credit card numbers, that can be verified and trusted by clients. In fact, some payment vendors such as PayPal, require SSL certificates to be used for customer transactions.

For instructions on how to use SSL certificates in your store, see our guides on [obtaining a commercially signed SSL certificate](/docs/guides/obtain-a-commercially-signed-tls-certificate/) and [using SSL certificates with Apache](/docs/guides/ssl-apache2-debian-ubuntu/).

{{< note respectIndent=false >}} Many payment vendors that require SSL do not support self-signed certificates. Depending on how you handle payments, it is likely you will need to purchase a commercially signed certificate.

When you [configure Apache to use the SSL certificate](/docs/guides/ssl-apache2-centos/#configure-apache-to-use-the-ssl-certificate), if you installed Magento in a subdirectory of your site, and only want that section to be encrypted, make sure to modify your `<VirtualHost *:443>` block to match. {{< /note >}}

Once you've installed your SSL certificate and configured Apache to serve your site securely, you'll need to configure Magento to use secure URLs.

1.  Log into your Magento admin panel using the URL you obtained after running the installation script.

2.  Click on **Stores** in the sidebar on the left of your screen, and select **Configuration** under the Settings submenu.

    ![Go to "Configuration" in the "Stores" menu.](magento-stores-config.png)

3.  On the next screen, you'll see a list of configuration settings including "General," "Catalog," and "Customers". Click on **General**, and select **Web** from its submenu.

    ![Select "Web" from the "General" menu.](magento-general-web.png)

4.  Select **Base URLs (Secure)**, which will open to display several text fields. Modify the first field, **Secure Base URL**, to use the HTTPS protocol, as opposed to HTTP:

    ```
    https://example.com
    ```

    ![Enter your domain with the HTTPS protocol.](magento-secure-base-url.png)

    This assumes that you used `/var/www/html/example.com/public_html` as your Magento installation directory, as we did in our examples. If you installed Magento in a subdirectory of your web root, you will need to modify that here. For example, if you installed Magento in a folder called `/var/www/html/example.com/public_html/magento`, you'll need to enter:

    ```
    https://example.com/magento
    ```

5.  Save your configuration.

6.  To verify that SSL has been configured correctly, visit your domain with the HTTPS protocol in a web browser and navigate to a few links. You should see a green lock icon in your browser's URL bar on each page, confirming that each page you (and your future customers) visit is secure.

## Next Steps

If you've followed this guide from the beginning, you should now have a secure, functional installation of Magento. You're now ready to configure your store and start selling products.

You may wish to install extensions to add functionality, or themes to change your site's user experience. If you do, be sure that the extensions and themes you purchase or install are compatible with Magento 2.

You may also want to configure caching with Varnish or other software to increase the speed of your site. For more information on this and other configuration options, see [Magento's configuration guide](https://devdocs.magento.com/guides/v2.2/config-guide/bk-config-guide.html).

Finally, be sure to keep your Magento software and its components up to date. Not only is this important to the security of your site, but it will also allow you to use the latest features and functions Magento has to offer. For more information, refer to the [Magento upgrade documentation](http://devdocs.magento.com/guides/v2.1/comp-mgr/bk-compman-upgrade-guide.html).
