# Setting Up Virtual Hosts (Recommended)
## SOURCE: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-18-04#step-5-%E2%80%94-setting-up-virtual-hosts-recommended)

### When using the Apache web server, you can use virtual hosts to encapsulate configuration details and host more than one domain from a single server. We will set up a domain called your_domain, but you should replace this with your own domain name. To learn more about setting up a domain name with DigitalOcean, see our Introduction to DigitalOcean DNS.

### Apache on Ubuntu 18.04 has one server block enabled by default that is configured to serve documents from the /var/www/html directory. While this works well for a single site, it can become unwieldy if you are hosting multiple sites. Instead of modifying /var/www/html, let’s create a directory structure within /var/www for a your_domain site, leaving /var/www/html in place as the default directory to be served if a client request doesn’t match any other sites.

### Create the directory for your_domain as follows:
    sudo mkdir /var/www/your_domain.com

### Next, assign ownership of the directory with the $USER environment variable:
    sudo chown -R $USER:$USER /var/www/your_domain.com

### The permissions of your web roots should be correct if you haven’t modified your unmask value, but you can make sure by typing:
    sudo chmod -R 755 /var/www/your_domain.com

### Next, create a sample index.html page using nano or your favorite editor:
    nano /var/www/your_domain.com/index.html

### Inside, add the following sample HTML:
    <html>
        <head>
            <title>Welcome to Your_domain!</title>
        </head>
        <body>
            <h1>Success!  The your_domain virtual host is working!</h1>
        </body>
    </html>
#### Save and close the file when you are finished.

### In order for Apache to serve this content, it’s necessary to create a virtual host file with the correct directives. Instead of modifying the default configuration file located at /etc/apache2/sites-available/000-default.conf directly, let’s make a new one at /etc/apache2/sites-available/your_domain.com.conf:
    sudo nano /etc/apache2/sites-available/your_domain.com.conf

### Paste in the following configuration block, which is similar to the default, but updated for our new directory and domain name:
    <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName your_domain.com
        ServerAlias www.your_domain.com
        DocumentRoot /var/www/your_domain.com
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>

### Notice that we’ve updated the DocumentRoot to our new directory and ServerAdmin to an email that the your_domain site administrator can access. We’ve also added two directives: ServerName, which establishes the base domain that should match for this virtual host definition, and ServerAlias, which defines further names that should match as if they were the base name.

#### Save and close the file when you are finished.

### Let’s enable the file with the a2ensite tool:
    sudo a2ensite your_domain.com.conf

### Disable the default site defined in 000-default.conf:
    sudo a2dissite 000-default.conf

### Next, let’s test for configuration errors:
    sudo apache2ctl configtest

### You should see the following output:
    Output
    Syntax OK

### Restart Apache to implement your changes:
    sudo systemctl restart apache2

### Apache should now be serving your domain name. You can test this by navigating to http://your_domain.com