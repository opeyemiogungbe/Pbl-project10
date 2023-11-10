# LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

What is an SSL/TLS Certificate?

  When a computer connects to a website, communication begins between the computer's web browser and the web server the site is hosted on. Typically, this communication is unguarded, meaning it's out in the open and any interested third party can have a look at it. if we're transmitting important personal information having it out in the open is not an ideal way to do things thus SSL/TLS certificate serve as a driver’s license of sorts—it serves two functions. It grants permissions to use encrypted communication via Public Key Infrastructure, and also authenticates the identity of the certificate’s holder.

  SSL Certificates facilitate an encrypted connection between a browser and a web server while also authenticating the identity of the website that owns the cert. With an SSL/TLS certificate, it's important to remember that the end user is the one visiting the website, but they are not the one who owns the certificate itself–that belongs to the company operating the website.

There are three kinds of certificates, DV, OV and EV. They offer varying levels of authentication but the same form of industry-standard encryption. The key to selecting the right SSL/TLS certificate is deciding what level of authentication you need. Smaller websites that do not collect user information may be better off saving money on a DV certificate. Business websites and E-Commerce sites should spring for an OV or EV certificate depending on their size and need for authentication.

In this project we will register our website with LetsEnrcypt Certificate Authority, to automate certificate issuance we will use a shell client recommended by LetsEncrypt - cetrbot.

### Task

This project consists of two parts:

* Configure Nginx as a Load Balancer
* Register a new domain name and configure secured connection using SSL/TLS certificates

### Part 1 - Configure Nginx As A Load Balancer

1. Instead of starting up a new ec2 instance, we will uninstall Apache from the existing Load Balancer server and install Nginx.

![Screenshot 2023-09-05 052052](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/2f3decae-b6de-4b62-9035-e46eb66853bd)

2. Update /etc/hosts file for local DNS with Web Servers' names (e.g. Web1 and Web2) and their local IP addresses

![Screenshot 2023-09-05 103940](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/2144ffb0-b30d-4ca2-96c7-6c34e111fed3)

3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

Update the instance and Install Nginx Install Nginx

```
sudo apt update
sudo apt install nginx
```

![Screenshot 2023-09-05 052530](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/4e119f1d-41b3-4ff4-8bd7-d870ec68e888)

Configure Nginx LB using Web Servers' names defined in /etc/hosts

Open the default nginx configuration file

`sudo vi /etc/nginx/nginx.conf`

```
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
#       include /etc/nginx/sites-enabled/*;
```

![Screenshot 2023-09-05 104015](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/ad8a940c-f798-475c-a1a7-4b316d44fbd1)

Restart Nginx and make sure the service is up and running

```
sudo systemctl restart nginx
sudo systemctl status nginx
```
### Part 2 - Register a new domain name and configure secured connection using SSL/TLS certificates

We will make necessary configurations to make connections to our Tooling Web Solution secured, to get a valid SSL certificate - you need to register a new domain name using any Domain name registrar (  Godaddy.com, Domain.com, Bluehost.com.)

1. We will register a new domain name with any registrar of your choice in any domain zone (e.g. .com, .net, .org, .edu, .info, .xyz or any other). We got one called `devtooling.space`

2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP ( this will help us to get a static IP address that doesn't change even after reboot)

![Screenshot 2023-09-05 105636](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/838dd771-109f-4280-8268-0edeccefd319)

3. Update A record in your registrar to point to Nginx LB using Elastic IP address

First we created a Hosted Zone for our Domain name in our AWS Route53

![Screenshot 2023-09-05 065522](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/da20e526-e0a5-467e-bfc6-211f14ec1f14)


Then we route the trafic to our Domain name registrar

![Screenshot 2023-09-05 074709](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/22c4941d-5e0b-440a-b012-ac19b6f6a1de)

Then we created a record with our Nginx LB static IP Address 

![Screenshot 2023-09-05 075736](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/83127432-0395-459e-8c59-3349d67ae7ee)

![Screenshot 2023-09-05 110337](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/06f7df0c-bc25-406e-84ef-6179bc095217)

Now we are going to check if our Web Servers can be reached from our browser using new domain name using HTTP protocol - http://<your-domain-name.com>

![Screenshot 2023-09-05 112424](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/7558bb09-b220-4248-b058-f0b91c169a70)

The image above shows us our new domain name ( devtooling.space) can be reached but not secured yet.

4. We will configure Nginx to recognize our new domain name

Updating our nginx.conf with server_name www.<our-domain-name.com> instead of server_name www.domain.com

![Screenshot 2023-09-05 104015](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/fe73c58c-40aa-4d7b-ab3a-6bdcfd2c7b76)

5. Install certbot and request for an SSL/TLS certificate

Note We must make sure snapd service is active and running

```
sudo systemctl status snapd
```
![Screenshot 2023-09-05 111332](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/c917182e-a272-435c-9630-53c60dc98aad)

Install certbot

```
sudo snap install --classic certbot
```
Now we will request our certificate by following the certbot instructions - Note: we will need to choose which domain we want our certificate to be issued for, domain name will be looked up from nginx.conf.

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```
![Screenshot 2023-09-06 062250](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/5cfd6398-2812-4249-b5ff-58acd623cc13)

Now we will test secured access to our Web Solution by trying to reach https://<your-domain-name.com>

![Screenshot 2023-09-06 070943](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/217b20e3-89a6-4470-a744-97c435aba0a3)

![Screenshot 2023-09-06 071211](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/d37c558a-fc06-4dc7-b497-c53088c866b1)

Now we are able to access our website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in our browser's search string.
Clicking on the padlock icon we can see the details of the certificate issued for our website in the second image.

6. We will set up periodical renewal of our SSL/TLS certificate

By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.
we can test renewal command in dry-run mode

```
sudo certbot renew --dry-run
```
![Screenshot 2023-09-06 073801](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/94c4e2b2-a422-4962-9b7a-a181059a13f2)

Best pracice is to have a scheduled job that run renew command periodically. Let us configure a cronjob to run the command twice a day.
To do so, lets edit the crontab file with the following command:

```
crontab -e
```

Add following line:

```
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```

![Screenshot 2023-09-06 074032](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/c1291427-3750-4e4a-b16b-23681a6127d5)

We can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.

We have just implemented an Nginx Load Balancing Web Solution with secured HTTPS connection with periodically updated SSL/TLS certificates.

Our target architecture will look like this:

![Screenshot 2023-11-09 190727](https://github.com/opeyemiogungbe/Pbl-project9/assets/136735745/2c786891-65ed-418c-91bc-f668fe5439a9)

