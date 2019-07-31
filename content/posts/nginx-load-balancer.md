---
title: "Nginx Load Balancer"
date: 2019-07-31T15:34:34+03:00
draft: fase
toc: false
images:
tags:
  - nginx
  - ubuntu
  - loadbalancer
---

Hey, everyone! How's it going? Today, We will setup **Nginx** with **Load Balancer**.

First of all, We should set ubuntu virtual machine. I'm gonna use Vagrant for this step.

```
vagrant init bento/ubuntu-18.04
vagrant up
```
This commands will create a virtual machine and boot it for us. This may take a while if you have not ubuntu box.

Then you can connect the virtual machine with `vagrant ssh` command. Now, we can install Nginx to our machine.
```
sudo apt-get update
sudo apt-get install nginx
```

You can check your Nginx server with that command:
```
curl localhost:80
```

## Create Server Blocks
Now, We can create our Server Blocks which means Virtual Hosts.
```
sudo mkdir -p /var/www/site1.com/html
sudo mkdir -p /var/www/site2.com/html
sudo mkdir -p /var/www/site3.com/html

sudo chown -R $USER:$USER /var/www/site1.com/html
sudo chown -R $USER:$USER /var/www/site2.com/html
sudo chown -R $USER:$USER /var/www/site3.com/html

sudo chmod -R 755 /var/www

sudo touch /var/www/site1.com/html/index.html && echo "Response from Site1" | sudo tee /var/www/site1.com/html/index.html
sudo touch /var/www/site2.com/html/index.html && echo "Response from Site2" | sudo tee /var/www/site2.com/html/index.html
sudo touch /var/www/site3.com/html/index.html && echo "Response from Site3" | sudo tee /var/www/site3.com/html/index.html
```

It's all done. We should create server block config files.
```
sudo vi /etc/nginx/sites-available/site1.com

# listen 8081 for Site1
# listen 8082 for Site2
# listen 8083 for Site3

server {
        listen 8081;
        listen [::]:8081;

        root /var/www/site1.com/html;
        index index.html index.htm index.nginx-debian.html;

        server_name site1.com www.site1.com;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

Now, It's time to enable our server block and restart nginx.
```
sudo ln -s /etc/nginx/sites-available/site1.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/site2.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/site3.com /etc/nginx/sites-enabled/
```

## Edit nginx.conf
```
sudo vi /etc/nginx/nginx.conf
```
Within the file, find the `server_names_hash_bucket_size` directive. Remove the # symbol to uncomment the line

```
sudo nginx -t
sudo systemctl restart nginx
```

## Create Load Balancer with Round Robin Strategy
We are gonna create a load balancer on port 8080.
```
sudo vi /etc/nginx/conf.d/load-balancer.conf

upstream balancer {
    server localhost:8081; 
    server localhost:8082; 
    server localhost:8083; 
}

server {
    listen 8080; 

    location / {
            proxy_pass http://balancer;
    }
}
```
Restart Nginx.

You should edit `Vagrantfile`. Find `config.vm.network "public_network"` and remove the # symbol then restart the vagrant box.

To find the public IP of the vagrant box, you should execute ifconfig in the virtual machine.

Here is a bash script for testing the load balancer
```
#!/bin/bash

for i in {1..100}
do
  curl 'http://[public_ip]:8080'
done

Output:
Response from Site1
Response from Site1
Response from Site2
Response from Site2
Response from Site3
Response from Site3
Response from Site1
Response from Site1
Response from Site2
Response from Site2
Response from Site3
Response from Site3
Response from Site1
Response from Site1
Response from Site2
Response from Site2
```

## Conclusion
As you can see in the output, there are different responses. Load balancers are useful when it comes to high traffic. I hope, this article would be useful to you. Thanks for reading :)
