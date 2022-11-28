## LOAD BALANCER SOLUTION WITH APACHE

- Load balancing is the process of redistribution of network traffic equally across a pool of resources that support an application thereby ensuring that no resouces either under/over utilized.

- In our setup in project 7, we had three web servers, each having their different public IPs and DNS names.  A client has to access them using diferent URLs. To hide this complexity, and to have a single point of access with a single public IP address/name, a Load Balancer can be used.

- A Load Balancer (LB) distributes clients’ requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.

![archy](https://user-images.githubusercontent.com/114196715/200878766-d8b94a01-1827-4899-95da-97497e7d92b5.png)

---

In this project the Tooling Website will be enhanced by adding a Load Balancer (Apache) to disctribute traffic between Web Servers and allow users to access our website using a single URL.

REQUIREMENTS

1. Two RHEL8 Web Servers (already configured in project 7)
2. One MySQL DB Server (based on Ubuntu 20.04)
3. One RHEL8 NFS server

STEP 1: CONFIGURE APACHE AS A LOAD BALANCER

* Create an Ubuntu Server 20.04 EC2 instance and name it LB_server to host apache which we shall use to load balance.

* Open TCP port 80 on LB_server instance by creating an Inbound Rule in Security Group.

* Install Apache Load Balancer on LB_server and configure it to point traffic coming to LB to both Web Servers: RHEL_7 and RHEL_8.

```
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2

```

* Ensure Apache is up and running; 

> ` sudo systemctl status apache2 `

* Configure load balancing

- Open up the default configuration file in Apache sites-available directory 

> `sudo vi /etc/apache2/sites-available/000-default.conf ` 

- Add the configuration below to the virtualhost section: Add the private IP of your individual web servers where necessary.

```
	  <Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests

        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

```

* Save the changes and restart apache2 ; 

> ` sudo systemctl restart apache2 `

* Verify that our confiuration works by accessing the public IP of our LB server in a browser.

* In order to test the load balancing action of our apache server, we shall check the access logs of each of the individual server; RHEL_7 and RHEL_8.

- To start with , we shall unmount /var/log/httpd/ from out Web Servers to the NFS server thus making sure that each web server has its own log direcotry.

-  refresh the public IP page of the LB_server in a web browser several times and make sure that both servers (RHEL_7 and RHEL_8) receive HTTP GET requests from your LB – new records must appear in each server’s log file.

![lb](https://user-images.githubusercontent.com/114786664/204325266-00c94233-fcc8-4ab0-929c-43443d796805.png)

---

![lb_2](https://user-images.githubusercontent.com/114786664/204325271-10944bed-feb4-4655-8f03-2524595ee284.png)

---

__END__
