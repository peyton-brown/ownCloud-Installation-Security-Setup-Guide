# Setting Up Virtual Hosts (Recommended)
## SOURCE: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-18-04#step-5-%E2%80%94-setting-up-virtual-hosts-recommended)

When using the Apache web server, you can use virtual hosts to encapsulate configuration details and host more than one domain from a single server. We will set up a domain called your_domain, but you should replace this with your own domain name. To learn more about setting up a domain name with DigitalOcean, see our Introduction to DigitalOcean DNS.

Apache on Ubuntu 18.04 has one server block enabled by default that is configured to serve documents from the /var/www/html directory. While this works well for a single site, it can become unwieldy if you are hosting multiple sites. Instead of modifying /var/www/html, let’s create a directory structure within /var/www for a your_domain site, leaving /var/www/html in place as the default directory to be served if a client request doesn’t match any other sites.

Create the directory for your_domain as follows:
    <code>sudo mkdir /var/www/your_domain</code>