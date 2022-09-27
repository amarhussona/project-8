# Project 8: Load Balancer Solution With Apache

## Configure Apache As A Load Balancer

First create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb, so your EC2 list will look like this:

![instances](./images/01_instances_setup.png)

Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group:

![inbound rule](./images/02_inbound_rule.png)

Next is to install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:

Install Apache:

`sudo apt update`
`sudo apt install apache2 -y`
`sudo apt-get install libxml2-dev`

![install apache2](./images/03_install_apache2.png)

Enable modules:

`sudo a2enmod rewrite`
`sudo a2enmod proxy`
`sudo a2enmod proxy_balancer`
`sudo a2enmod proxy_http`
`sudo a2enmod headers`
`sudo a2enmod lbmethod_bytraffic`

![enable modules](./images/04_enable_modules.png)

Restart apache2 service:

`sudo systemctl restart apache2`

`sudo systemctl status apache2`

![restart apache2](./images/05_restart_apache_ensure_active.png)

Configure load balancing:

`sudo vi /etc/apache2/sites-available/000-default.conf`

Add this configuration into this section <VirtualHost *:80> :

<!-- <Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/ -->

![configure load balancing](./images/07_Configure_load_balancing.png)

Restart apache server:

`sudo systemctl restart apache2`

Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:

`http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php`

![verify](./images/08_loadbalancer_ip.png)

Open two ssh/Putty consoles for both Web Servers and run following command:

`sudo tail -f /var/log/httpd/access_log`

![access log](./images/09_access_log.png)

Try to refresh your browser page http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file as shown in the image:

![updated access log](./images/10_updated_access_log.png)

## Configure Local DNS Names Resolution

Configure the file /etc/hosts:

`sudo vi /etc/hosts`

Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> web2
<WebServer2-Private-IP-Address> web3

![configure local DNS](./images/11_add_ips_to_etchosts.png)

Now you can update your LB config file with those names instead of IP addresses:

BalancerMember http://web2:80 loadfactor=5 timeout=1
BalancerMember http://web3:80 loadfactor=5 timeout=1

![update LB config file](./images/12_update_hostname_apache_config.png)

Try to curl your Web Servers from LB locally 'curl http://web2 or curl http://web3 – it shall work:

![curl](./images/13_curl_web2.png)
![curl](./images/13_curl_web3.png)

This is only a local configuration

