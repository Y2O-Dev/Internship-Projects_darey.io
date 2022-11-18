# LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS CERTIFICATE

Load balancing also refer to as a **server farm** or **server pool** refers to efficiently distributing incoming network traffic across a group of backend servers. A load balancer acts as a police that direct incoming traffic to a specific server that can fulfil the client request, thereby preventing particular server from being overworked.

SSL stands for Secure Sockets Layer which is the standard technology for keeping an internet connection secure and safeguarding any sensitive data that is being sent over the internet. It does this via encryption algorithm. TLS (Transport Layer Security) is the updated, more secure, version of SSL.

### Parts

> There is two parts to this project:
> 1. Configure Nginx as a Load Balancer
> 2. Register a new domain name and configure secured connection using SSL/TLS certificate

The architecture diagram should look like this:

![architecture_diag](https://user-images.githubusercontent.com/114786664/202806045-2e85cdd9-c28b-44b3-a1e2-127a19074bde.png)

---

### Part 1
1. Create an EC2 VM based on Ubuntu Server 20.04 LTS.
- Update the instance and Install Nginx with the below command:
```
sudo apt update &&
sudo apt install nginx
```
2. Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses using:

> ` sudo  vi /etc/hosts`

![host](https://user-images.githubusercontent.com/114786664/202806236-3ce19f68-66f0-4e23-bc64-f3bbeb557712.png)


3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers.

- Open and modify the default nginx configuration file with the below code:

> `sudo vi /etc/nginx/nginx.conf `

- Enter the below code in the opened file.

![conf_code](https://user-images.githubusercontent.com/114786664/202808087-3da0ebf3-b7f2-403d-8e36-3d0fda1f6ec9.png)

- Restart Nginx and make sure the service is up and running

```
sudo systemctl restart nginx
sudo systemctl status nginx
```
- Test if the load balancer is functioning

![l_balancing](https://user-images.githubusercontent.com/114786664/202806551-f85204ad-830f-4698-b8f9-b697ef1fa409.png)

---

### Part 2: Register a new domain name and configure secured connection using SSL/TLS certificate

- A new domain name (yakubyo-cloudresume.gq) was created.

- Update A record in your registrar to point to Nginx LB using Elastic IP address  as shown in the below diagram

![Screenshot from 2022-11-16 16-34-29a record](https://user-images.githubusercontent.com/114786664/202806909-c3434815-3529-47f9-b967-283d755c2be0.png)

- Configure Nginx to recognize your new domain name

![etc](https://user-images.githubusercontent.com/114786664/202807002-155c2bac-c657-45fc-9d59-56f0589e5da9.png)

![lb_server](https://user-images.githubusercontent.com/114786664/202807252-2dd476a7-c52a-490a-8d44-17e112c24f27.png)


- Install certbot and request for an SSL/TLS certificate

> Ensure snapd service is active and running, then 
> Install certbot
```
sudo systemctl status snapd 
sudo snap install --classic certbot
```

- Request your certificate by following the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from *nginx.conf file*

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```

![certbot](https://user-images.githubusercontent.com/114786664/202807558-0d8d333f-a825-4d47-962a-77bb687dc4ed.png)

- Check if the domain name is accessible via https:


![https](https://user-images.githubusercontent.com/114786664/202807926-3cc8f03f-8aaa-48ab-ae8c-fcb4e5a76d0c.png)


- Set up periodical renewal of your SSL/TLS certificate by:
i- Dry running: Manual approach using the following code:

> ` sudo certbot renew --dry-run`

![cert_status](https://user-images.githubusercontent.com/114786664/202807364-bbf81409-4e15-4b3a-b777-1626e8275ca2.png)

ii-  Scheduled running with Cron-job using the below code:

> ` crontab -e`

- Enter the following line:
```
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```

---
__END__