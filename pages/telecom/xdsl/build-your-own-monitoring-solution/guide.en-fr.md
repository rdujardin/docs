---
title: 'Build your own monitoring solution'
excerpt: 'Build your own monitoring solution using Insight and Grafana'
section: Diagnostic et dépannage
---

**Last updated on 12/01/2020 2020**

## Goal

OVH offers with its Insight solution to share the metrics of your services to integrate them into your own solutions.

**Find out how to activate Insight and set up your first Grafana dashboard to monitor your Internet access.**

## Requirements

* Have an [xDSL or OVHcloud fiber internet access] (https://www.ovhtelecom.fr/offre-internet/){.external};
* The remote configuration of your modem must be enabled.

## In practice

## Step 1 : Get your OVH Insight metric reading token

1. Go to the APIv6 console at this address : [https://api.ovh.com/console/](https://api.ovh.com/console/){.external}

2. Authenticate with your OVHcloud credentials

3. Execute the following method to retrieve your read token :

> [!api]
>
> @api {GET} /me/insight
>

![getToken](images/token.png){.thumbnail}

4. Securely save your returned reading token under the key `access`.

## Step 2 : Set up your first Grafana dashboard

1. Go to the provided Grafana instance at this address : [https://grafana.metrics.ovh.net](https://grafana.metrics.ovh.net){.external}

2. Authenticate with your OVH credentials

3. Choose `Add data source`{.action}

![grafanaAddSource](images/grafana1.png){.thumbnail}

4. Configure the data source such as :
    ```
    Name : Insight Warp10
    Type : Warp10
    Url  : https://warp10.insight.eu.metrics.ovh.net
    ```

5. Add a constant named `token` using your OVH Insight read token

![grafanaAddConstant](images/grafana2.png){.thumbnail}

6. Click on `Add`{.action} to finalize

7. Click on the Grafana icon at the top left and choose from the menu: `Dashboard`{.action}, then `Import`{.action}

8. Upload the following template: connectivityIspAccess.json TODO

9. Click on `Import`{.action} to finalize


> [!primary]
>
> Congratulations ! You now have your first Grafana dashboard with the following Internet access informations :
> - Incoming traffic
> - Outgoing traffic
>
> And if your modem is compatible :
> - Synchronisation, SNR *(Signal to Noise Ratio)*, Attenuation and CRC errors
> - Synchronisation, connection and modem uptime
>
