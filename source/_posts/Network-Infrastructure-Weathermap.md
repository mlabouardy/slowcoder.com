---
title: Network Infrastructure Weathermap
date: 2018-03-26 12:59:57
tags:
- DevOps
- Infrastructure
- Monitoring
categories:
- DevOps
cover_index: /assets/Network-Infrastructure-Weathermap-Index.png
---
The main goal of collecting metrics is to store them for long term usage and to create graphs to debug problems or identify trends. However, monitoring your system isn’t enough to identity the problem’s & anomalies root cause. It’s necessary to have a high-level overview of your network **backbone**. **Weathermap** is perfect for a **Network Operations Center** (**NOC**). In this post, I will show you how to build one using Open Source tools only.

![](https://cdn-images-1.medium.com/max/1000/0*A6E25joDjeD7uEqs.)

**Icinga 2** will collect metrics about your backbone, write checks results metrics and performance data to **InfluxDB** (supported since [Icinga 2.5](https://www.icinga.com/2016/08/31/icinga-2-meets-influxdb/)). Visualize these metrics in **Grafana** in map form.

To get started, add your desired host configuration inside the *hosts.conf* file:

{% gist 5481342c21dc8ffed9a5a5ac5d870f11 hosts.conf %}

Note: the *city* & *country* attributes will be used to create the weathermap.

To enable the **InfluxDBWriter** on your Icinga 2 installation, type the following command:

{% codeblock %}
icinga2 feature enable influxdb
{% endcodeblock %}

Configure your **InfluxDB** host and database in */etc/icinga2/features-enabled/influxdb.conf (*[Learn more about the InfluxDB configuration](https://www.icinga.com/docs/icinga2/latest/doc/09-object-types/)*)*

{% gist 432ebd020745338fdd8487b6c07e0776 influxdb.conf %}

**Icinga 2** will forward all your metrics to a *icinga2_metrics* database. The included *host* and *service* templates define a storage, the *measurement* represents a table by which metrics are grouped with tags certain measurements of certain hosts or services are defined (notice the *city* & *country* tags usage).

Don’t forget to restart **Icinga 2** after saving your changes:

{% codeblock %}
service icinga2 restart
{% endcodeblock %}

Once **Icinga 2** is up and running it’ll start collecting data and writing them to **InfluxDB**:

![](https://cdn-images-1.medium.com/max/800/0*CcFdpVNE9JxWnD8v.)

Once our data arrived, it’s time for visualization. **Grafana** is widely used to generate graphs and dashboards. To create a **Weathermap** we can use a Grafana plugin called [Worldmap Panel (https://github.com/grafana/worldmap-panel). Make sure to install it using *grafana-cli* tool:

{% codeblock %}
grafana-cli plugins install grafana-worldmap-panel
{% endcodeblock %}

The plugin will be installed into your grafana plugins directory (*/var/lib/grafana/plugins*)

Restart **Grafana**, navigate to Grafana web interface and create a new *datasource*:

![](https://cdn-images-1.medium.com/max/800/0*8_lrv5xqMS43JM8a.)

Create a new Dashboard:

![](https://cdn-images-1.medium.com/max/800/0*2AxYUc5UdVF-iXwD.)

The **Group By** clause should be the **country** code and an **alias** is needed too. The alias should be in the form **$tag_field_name**. See the image below for an example of a query:

![](https://cdn-images-1.medium.com/max/800/0*zQoTIn9EdNtmmjh-.)

Under the **Worldmap** tab, choose the **countries** option:

![](https://cdn-images-1.medium.com/max/800/0*A7krU_fxs1kyNg0m.)

Finally, you should see a tile map of the world with circles representing the state of each host.

![](https://cdn-images-1.medium.com/max/800/0*K4zTlL-Z-19E9mRa.)

The field *state *possible values (**0** — OK, **1** — Warning, **2** — Critical, **3** — Unknown/Unreachable)

Note: For lazy people I created a ready to use Dashboard you can import from [GitHub](https://github.com/mlabouardy/grafana-dashboards/tree/master/icinga2).