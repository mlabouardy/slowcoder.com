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
---

Being able to access **AWS** resources directly in secure way can be very useful. To achieve this you can:

* Setup a dedicated connection with **AWS Direct Connect**
* Use a **Network Appliance**
* Use a** Software Defined Private Network** like **OpenVPN**

In this post, I will walk you through how to create an **OpenVPN** server on AWS, to connect securely to your VPC, Private Network resources and applications from any device anywhere.

To get started, sign in to your **AWS Management Console** and launch an **EC2 instance** from the **OpenVPN Access Server AWS Marketplace **offering:

![](https://cdn-images-1.medium.com/max/1000/0*RxIGQWMzZ4CddqT0.)

For demo purpose, choose *t2.micro:*

![](https://cdn-images-1.medium.com/max/1000/0*wPgYMoC0t73N4aoY.)

Use the default settings with the exception of “**Enable termination protection**” as we dont want our **VPN** being terminated on accident:

![](https://cdn-images-1.medium.com/max/1000/0*x3v8Khtz7PCZ8f3H.)

Assign a new **Security Group** as below:

![](https://cdn-images-1.medium.com/max/1000/0*LsR4IAfGXXfuIBcV.)

* **TCP - 22** : Remote access to the instance.
* **TCP - 443** : HTTPS, this is the interface used by users to log on to the VPN server and retrieve their keying and installation information.
* **TCP - 943** : OpenVPN Admin Web Dashboard.
* **UDP - 1194** : OpenVPN UDP Port.

To ensure our VPN instance Public IP address doesnt change if it’s stopped, assign to it an **Elastic IP**:

![](https://cdn-images-1.medium.com/max/1000/0*tjptbp-B9P0YO2Tw.)

For simplicity, I added an **A** record in **Route 53** which points to the instance **Elastic IP**:

![](https://cdn-images-1.medium.com/max/1000/0*gyYVtEyL0GdB3Goz.)

Once the **AMI **is successfully launched, you will need to connect to the server via **SSH** using the **DNS** record:

{% codeblock %}
ssh openvpnas@openvpn.slowcoder.com -i /path/to/key.pem
{% endcodeblock %}

On first time connecting, you will be prompted and asked to setup the OpenVPN server:

![](https://cdn-images-1.medium.com/max/1000/0*UaM_RebtMcLorGPS.)

Setup a new password for the *openvpn* admin user:

{% codeblock %}
sudo passwd openvpn
{% endcodeblock %}

Point your browser to [https://openvpn.slowcoder.com](https://openvpn.slowcoder.com/), and login using *openvpn* credentials

![](https://cdn-images-1.medium.com/max/1000/0*8lAgBisfmqFeOGN4.)

Download the **OpenVPN Connect Client**, after your installation is complete, click on “**Import**” then “**From server**” :

![](https://cdn-images-1.medium.com/max/1000/0*8gJhBE7WkluqbUL1.)

Then, type the **OpenVN DNS** name:

![](https://cdn-images-1.medium.com/max/1000/0*7cYXugAKr1dXH4hE.)

Enter your *openvpn* as the username and enter the same password as before and click on “**connect**“:

![](https://cdn-images-1.medium.com/max/1000/0*aoo23Jib_en5KKVg.)

After you are connected, you should see a green check mark:

![](https://cdn-images-1.medium.com/max/1000/0*6m-T7uNOkOZPV9nF.)

To verify the client is connected, login to **OpenVPN Admin Dashboard** on [https://openvpn.slowcoder.com/admin](https://openvpn.slowcoder.com/admin) :

![](https://cdn-images-1.medium.com/max/1000/0*X3JvgFjbZcg30GUp.)

Finally, create a simple web server instance in a private subnet to verify the VPN is working:

![](https://cdn-images-1.medium.com/max/1000/0*xwkYZU_j3PIkzKSY.)

If you point your browser to the webserver private address, you should see a simple HTML page:

![](https://cdn-images-1.medium.com/max/1000/0*uNaD3HvXcuz4576Y.)