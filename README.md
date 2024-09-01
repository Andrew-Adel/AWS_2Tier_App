![Screenshot from 2024-09-01 02-39-24](https://github.com/user-attachments/assets/66f768b2-ee7d-46f4-a45b-1ca73175ddfd)# AWS_2Tier_App
## about

## Diagram

## Create My Network "CIDR and its subnet" 
### 1. Create VPC
1. Go to the VPC Dashboard in AWS Management Console.
2. Click on "Create VPC".
3. Choose the VPC-only option, and set the CIDR block to 192.168.0.0/16.
4. Name the VPC, '`Lec1-Assignment-2-Teir-App`'.

### 2. Create 4 subnets 
1. 2 public (Lec1-Assignment-Public-Subnet-1 & Lec1-Assignment-Public-Subnet-2)
2. 2 private (Lec1-Assignment-Private-Subnet-1 & Lec1-Assignment-Private-Subnet-2)

### 3. Create Internet Gateway
create interent gateway `Lec1-Assignment-IGW` to make public subnet access interent 'inbound and outbound'

### 4. create Nat-Gateway
create interent gateway `My-Nat-Gateway` to make private subnet access interent 'outbound' only, for download and get its package and tools
this nat-gateway exist in subnet `Lec1-Assignment-Private-Subnet-2`

### 5. Make Route Tables
1. first route table to connect (Lec1-Assignment-Public-Subnet-1 & Lec1-Assignment-Public-Subnet-2) with `Lec1-Assignment-IGW`
2. second route table to connect (Lec1-Assignment-Private-Subnet-1 & Lec1-Assignment-Private-Subnet-2) with `My-Nat-Gateway`

### Image of Network
![Screenshot from 2024-09-01 02-35-22](https://github.com/user-attachments/assets/9043d0cb-6bdf-439d-9f3b-feed57e809db)

## Create Public Instances
1. create 2 public instances (Lec1-Assignment-instance-A & Lec1-Assignment-instance-B)
  each one have elastic public IP to be access to/by the interent easily and can connect using SSH with key-pair
  each one contain private id too, to be connect with other instance inside the network 
  it contain nginx that run as a forwarding to the private proxy instance which is the engering of teir 2 
![Screenshot from 2024-09-01 02-39-27](https://github.com/user-attachments/assets/03e12a8d-26c6-4454-80ae-8d33c309cd45)
![Screenshot from 2024-09-01 02-39-24](https://github.com/user-attachments/assets/4413449a-b5cc-4491-b4fa-b1b7cded3e5f)

2. create public proxy instance (Lec1-Assignment-Public-Proxy)
  it has elastic public IP to be access to/by the interent easily and can connect using SSH with key-pair
  it contain nginx that run as a load balance between the 2 public instance
![Screenshot from 2024-09-01 02-38-54](https://github.com/user-attachments/assets/65eecb18-5d6a-4c0d-9feb-66565da7c4c7)
### test of public proxy with the 2 public instances
![Screenshot from 2024-09-01 02-33-58](https://github.com/user-attachments/assets/f407ad50-a1e6-4cfe-abc6-3193532ce017)
![Screenshot from 2024-09-01 02-33-54](https://github.com/user-attachments/assets/9f097381-3ab8-4671-99b9-bb2758d9a0d3)


## Create Private Instances

1. create 2 private instances (Lec1-Assignment-instance-C & Lec1-Assignment-instance-D)
  each one have only private IP to be connect with other instance inside the network and can connect using SSH with key-pair
![Screenshot from 2024-09-01 02-39-06](https://github.com/user-attachments/assets/555b3265-d9bd-4328-904e-5164a9a2e156)
![Screenshot from 2024-09-01 02-38-59](https://github.com/user-attachments/assets/7f738f5c-662f-48d4-9290-3dbd2e78f150)


2. create private proxy instance (Lec1-Assignment-Private-Proxy)
  it has only private IP to be connect with other instance inside the network and can connect using SSH with key-pair
  it contain nginx that run as a load balance between the 2 private instance
![Screenshot from 2024-09-01 02-39-10](https://github.com/user-attachments/assets/90118f83-1914-4caa-92fc-fc5e65eed576)
### test of public proxy to the 2 public instances then to private proxy finally to the 2 private instances
![Screenshot from 2024-09-01 02-33-54](https://github.com/user-attachments/assets/41755536-7958-4279-812c-d1fe251578e7)
![Screenshot from 2024-09-01 02-33-58](https://github.com/user-attachments/assets/920232a1-f559-4cc6-a9c7-bfc28b5efc32)

## all instance
![Screenshot from 2024-09-01 02-38-39](https://github.com/user-attachments/assets/53df9be9-09a3-4d19-bccb-9155fc7dc83f)


## config inside each instance

### for public proxy instance
#### security group
Allow inbound traffic:
    Type: HTTP, Protocol: TCP, Port Range: 80, Source: Anywhere (0.0.0.0/0, ::/0).
    Type: SSH, Protocol: TCP, Port Range: 22, Source: My IP (for secure access)// or anywhere.
#### configuration inside instance
1. connect using SSH into each instance using the public IP address:
```bash
ssh -i 2-Teir-App.pem ec2-user@<public-ip-od-instance>
```
2. Install NGINX:
```bash
sudo yum update -y
sudo yum install nginx1 -y
sudo systemctl start nginx
sudo systemctl enable nginx
```
3. config the load balance
add following to `sudo vi /etc/nginx/nginx.conf
`
```bash
http {
    upstream backend {
        server <public-instance-1-private-ip>:80;
        server <public-instance-2-private-ip>:80;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```
check the syntax error
```bash
sudo nginx -t
```
apply the configuration by reload nginx
```bash
sudo systemctl reload nginx
```


### for privaty proxy instance
#### security group
Allow inbound traffic:
    Type: HTTP, Protocol: TCP, Port Range: 80, Source: Anywhere (0.0.0.0/0, ::/0).
    Type: SSH, Protocol: TCP, Port Range: 22, Source: My IP (for secure access)// or anywhere.
#### configuration inside instance
0. copy key-pem to a public instance
```bash
scp -i 2-Teir-App.pem 2-Teir-App.pem ec2-user@34.200.246.143:/home/ec2-user/
```
2. connect using SSH into each instance using the public IP address:
```bash
ssh -i 2-Teir-App.pem ec2-user@<public-ip-of-any-public-instance-has-the-key>
ssh -i 2-Teir-App.pem ec2-user@<public-ip-of-private-instance>
```
2. Install NGINX:
```bash
sudo yum update -y
sudo yum install nginx1 -y
sudo systemctl start nginx
sudo systemctl enable nginx
```
3. config the load balance
add following to `sudo vi /etc/nginx/nginx.conf
`
```bash
http {
    upstream backend {
        server <public-instance-1-private-ip>:80;
        server <public-instance-2-private-ip>:80;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```
check the syntax error
```bash
sudo nginx -t
```
apply the configuration by reload nginx
```bash
sudo systemctl reload nginx
```


### for public instance
#### security group
Allow inbound traffic:
    Type: HTTP, Protocol: TCP, Port Range: 80, Source: Anywhere (0.0.0.0/0, ::/0).
    Type: SSH, Protocol: TCP, Port Range: 22, Source: My IP (for secure access)// or anywhere.
#### configuration inside instance
1. connect using SSH into each instance using the public IP address:
```bash
ssh -i 2-Teir-App.pem ec2-user@<public-ip-of-instance>
```
2. Install NGINX:
```bash
sudo yum update -y
sudo yum install nginx1 -y
sudo systemctl start nginx
sudo systemctl enable nginx
```
add following to `sudo vi /etc/nginx/conf.d/forwarding.conf`
```
server {
    listen 80;
    server_name _;

    location / {
        # Forward requests to the private proxy instance
        proxy_pass http://192.168.3.139;  
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    error_page 404 /404.html;
    location = /404.html {
    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
    }
}
```

add following to `sudo vi /etc/nginx/nginx.conf`
```bash
server {
    listen 80;
    server_name _;

    location / {
        # Forward requests to the private proxy instance
        proxy_pass http://192.168.3.139;  # Make sure this is the correct private IP
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    error_page 404 /404.html;
    location = /404.html {
    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
    }
}
```
check the syntax error
```bash
sudo nginx -t
```
apply the configuration by reload nginx
```bash
sudo systemctl reload nginx
```



### for private instance
add following to `sudo vi /usr/local/bin/update-ip.sh`
```bash
# Create an HTML page with the private IP address
echo "<!DOCTYPE html>" > /usr/share/nginx/html/index.html
echo "<html>" >> /usr/share/nginx/html/index.html
echo "<head><title>My Private EC2</title></head>" >> /usr/share/nginx/html/index.html
echo "<body>" >> /usr/share/nginx/html/index.html
echo "<h1>Private IP Address IN Private EC2 (Last Node)</h1>" >> /usr/share/nginx/html/index.html
echo "<p>Your private IP address is: <b>$PRIVATE_IP</b></p>" >> /usr/share/nginx/html/index.html

echo "</body>" >> /usr/share/nginx/html/index.html
echo "</html>" >> /usr/share/nginx/html/index.html
```

Run the Script Again:
Execute the updated script manually to verify that it works correctly:
```bash
sudo chmod +x /usr/local/bin/update-ip.sh
sudo /usr/local/bin/update-ip.sh
```
check the syntax error
```bash
sudo nginx -t
```
apply the configuration by reload nginx
```bash
sudo systemctl reload nginx
```
