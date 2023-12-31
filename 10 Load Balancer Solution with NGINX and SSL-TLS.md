# Load Balancer Solution with NGINX and SSL/TLS

By now we have learned what Load Balancing is used for and have configured an LB solution using Apache, but a DevOps engineer must be a versatile professional and know different alternative solutions for the same problem. That is why, in this project we will configure an Nginx Load Balancer solution.

It is also extremely important to ensure that connections to our Web solutions are secure and information is encrypted in transit – we will also cover connection over secured HTTP (HTTPS protocol), its purpose and what is required to implement it.

When data is moving between a client (browser) and a Web Server over the Internet – it passes through multiple network devices and, if the data is not encrypted, it can be relatively easy intercepted by someone who has access to the intermediate equipment. This kind of information security threat is called Man-In-The-Middle (MIMT) attack.

This threat is real – users that share sensitive information (bank details, social media access credentials, etc.) via non-secured channels, risk their data to be compromised and used by cybercriminals.

SSL and its newer version, TSL – is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server. Here we will refer this family of cryptographic protocols as SSL/TLS – even though SSL was replaced by TLS, the term is still being widely used.

SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.

There are different types of SSL/TLS certificates – you can learn more about them here. You can also watch a tutorial on how SSL works here or an additional resource here

In this project you will register your website with LetsEnrcypt Certificate Authority, to automate certificate issuance you will use a shell client recommended by LetsEncrypt – cetrbot.


### Task

This project consists of two parts:

Configure Nginx as a Load Balancer

Register a new domain name and configure secured connection using SSL/TLS certificates

Our target architecture will look like this:

![image](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/f1965df6-b724-4642-b58c-1dc2c2bd9793)


## CONFIGURE NGINX AS A LOAD BALANCER

Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx LB (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)
Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses

![setting up instance for project 10 for Nginx webserver](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/764ff631-4b12-430d-9ee5-abe9db93bde5)

![conect to the nginx server](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/c40c344e-0f17-4f58-a7e4-2cf5d34d79b7)

![opening port 80 and 443 for http and https traffic](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/b300b7ed-21df-4fb5-a5d3-8a7adf5ef7e9)


Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

Update the instance and Install Nginx

`sudo apt update`

`sudo apt install nginx`

Configure Nginx LB using Web Servers’ names defined in /etc/hosts

Open the default nginx configuration file

`sudo vi /etc/nginx/nginx.conf`

#insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#include /etc/nginx/sites-enabled/*;

![updating nginx conf file using vi to add our domain names](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/ad1a3d8a-8e93-492d-bcb5-3f5ef6c13371)

![linking loadbalancer conf to sites available](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/7a155225-291e-4b00-92d2-a592393be873)

![removing nginx default page](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/11bd7151-c81c-4820-8883-764a58f51c2c)


Restart Nginx and make sure the service is up and running

`sudo systemctl restart nginx`

`sudo systemctl status nginx`


## REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES

Let us make necessary configurations to make connections to our Tooling Web Solution secured!

In order to get a valid SSL certificate – you need to register a new domain name, you can do it using any Domain name registrar – a company that manages reservation of domain names. The most popular ones are: Godaddy.com, Domain.com, Bluehost.com.

Register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other)

![setting up a domain name using namecheap](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/ad3da26f-dce0-4358-8948-ae22bf495ead)

![adding name servers from AWS ROUTE53 to namecheap domain created](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/a3a96eae-e2bd-45d3-b254-cb4bd1bc87b0)

Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP

You might have noticed, that every time you restart or stop/start your EC2 instance – you get a new public IP address. When you want to associate your domain name – it is better to have a static IP address that does not change after reboot. Elastic IP is the solution for this problem, learn how to allocate an Elastic IP and associate it with an EC2 server on this page.

Update A record in your registrar to point to Nginx LB using Elastic IP address

![connecting hosted zones on Route 53](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/6264762b-a045-4080-a977-fac82ce9ee28)

![creating A record in route53 for our domain](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/7deb6d84-a756-4211-9150-58e5d464aa4d)

Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol – http://<your-domain-name.com>

![image](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/04eab50a-7e00-4848-8326-934b020aad70)

Configure Nginx to recognize your new domain name

Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com

Install certbot and request for an SSL/TLS certificate

Make sure snapd service is active and running

`sudo systemctl status snapd`

Install certbot

`sudo snap install --classic certbot`

![image](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/348066ed-09c9-4f7a-9062-54611c2fc8d1)


Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it on step 4).

`sudo ln -s /snap/bin/certbot /usr/bin/certbot`

`sudo certbot --nginx`

![image](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/de385ed5-d1a9-4a4f-8b6e-bfabb02d9412)


Test secured access to your Web Solution by trying to reach https://<your-domain-name.com>

You shall be able to access your website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in your browser’s search string.

![image](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/f4cd14a6-f2bb-4bc9-8809-2f669f7ef8a4)


Click on the padlock icon and you can see the details of the certificate issued for your website.  

Set up periodical renewal of your SSL/TLS certificate

By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

You can test renewal command in dry-run mode

`sudo certbot renew --dry-run`

![image](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/2773d8d7-405c-42dd-98a0-f3f42ddd460c)


Best practice is to have a scheduled job than to run renew command periodically. Let us configure a cronjob to run the command twice a day.

To do so, lets edit the crontab file with the following command:

`crontab -e`

![image](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/115783cf-a4e3-482b-9ea0-1b4197bffcdb)


Add following line:

`* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`

![image](https://github.com/Sakirat/Project_Based_Learning/assets/110112922/eda32c46-6577-4b70-a1b6-67b89b8c704d)

You can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.

Congratulations!

You have just implemented an Nginx Load Balancing Web Solution with secured HTTPS connection with periodically updated SSL/TLS certificates.

