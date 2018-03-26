---
title: AWS OpenVPN Access Server
date: 2018-03-26 12:59:48
tags:
- AWS
- Cloud
- Security
- VPC
categories:
- Cloud
cover_index: /assets/AWS-OpenVPN-Access-Server-Index.png
cover_detail: /assets/AWS-OpenVPN-Access-Server-Detail.jpg
---
Being able to access AWS resources directly in secure way can be very useful. To achieve this you can:

- Setup a dedicated connection with AWS Direct Connect
- Use a Network Appliance
- Use a Software Defined Private Network like OpenVPN

In this post, I will walk you through how to create an OpenVPN server on AWS, to connect securely to your VPC, Private Network resources and applications from any device anywhere.

To get started, sign in to your AWS Management Console and launch an EC2 instance from the OpenVPN Access Server AWS Marketplace offering:

For demo purpose, choose _t2.micro:_

Use the default settings with the exception of “Enable termination protection” as we dont want our VPN being terminated on accident:

Assign a new Security Group as below:

- TCP — 22 : Remote access to the instance.
- TCP — 443 : HTTPS, this is the interface used by users to log on to the VPN server and retrieve their keying and installation information.
- TCP — 943 : OpenVPN Admin Web Dashboard.
- UDP — 1194 : OpenVPN UDP Port.

To ensure our VPN instance Public IP address doesnt change if it’s stopped, assign to it an Elastic IP:

For simplicity, I added an A record in Route 53 which points to the instance Elastic IP:

Once the AMI is successfully launched, you will need to connect to the server via SSH using the DNS record:

ssh [openvpnas@openvpn.slowcoder.com](mailto:openvpnas@openvpn.slowcoder.com) -i /path/to/key.pem

On first time connecting, you will be prompted and asked to setup the OpenVPN server:

Setup a new password for the _openvpn_ admin user:

> sudo passwd openvpn

Point your browser to [https://openvpn.slowcoder.com](https://openvpn.slowcoder.com/), and login using _openvpn_ credentials

Download the OpenVPN Connect Client, after your installation is complete, click on “Import” then “From server” :

Then, type the OpenVN DNS name:

Enter your _openvpn_ as the username and enter the same password as before and click on “connect“:

After you are connected, you should see a green check mark:

To verify the client is connected, login to OpenVPN Admin Dashboard on [https://openvpn.slowcoder.com/admin](https://openvpn.slowcoder.com/admin) :

Finally, create a simple web server instance in a private subnet to verify the VPN is working:

If you point your browser to the webserver private address, you should see a simple HTML page:
