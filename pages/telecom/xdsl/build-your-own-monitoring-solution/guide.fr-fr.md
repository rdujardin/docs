---
title: 'Mettre en place votre propre solution de monitoring'
excerpt: 'Mettre en place votre propre solution de monitoring de vos accès Internet en utilisant Insight et Grafana'
section: Diagnostic et dépannage
---

**Dernière mise à jour le 01/12/2020**

## Objectif

OVH propose avec sa solution Insight de partager les métriques de vos services pour les intégrer à vos propres solutions.

**Découvrez comment activer Insight et mettre en place votre premier tableau de bord Grafana pour monitorer vos accès Internet.**

## Prérequis

* Disposer d’un [accès internet xDSL ou fibre OVHcloud](https://www.ovhtelecom.fr/offre-internet/){.external};
* La configuration à distance est de votre modem doit être activée.

## En pratique

### Etape 1 : Récupérer votre jeton de lecture de métrique OVH Insight

1. Rendez-vous sur la console APIv6 à cette adresse : [https://api.ovh.com/console/](https://api.ovh.com/console/){.external}

2. Authentifiez-vous avec vos indentifiants OVHcloud

3. Executez la méthode suivante pour récupérer votre jeton de lecture :

> [!api]
>
> @api {GET} /me/insight
>

![apiGetInsightToken](images/token.png){.thumbnail}

4. Enregistrez de manière sécurisé votre jeton de lecture retourné sous la clé `access`.

### Etape 2 : Mettre en place votre premier tableau de bord Grafana

1. Rendez-vous sur l'interface Grafana fournie par OVHcloud à l'adresse : [https://grafana.metrics.ovh.net](https://grafana.metrics.ovh.net){.external}

2. Authentifiez-vous avec vos indentifiants OVHcloud

3. Choisissez `Add data source`{.action}

![grafanaAddSource](images/grafana1.png){.thumbnail}

4. Configurez la source de donnée tel que :
    ```
    Name : Insight Warp10
    Type : Warp10
    Url  : https://warp10.insight.eu.metrics.ovh.net
    ```

5. Ajoutez une constante nommée `token` en utilisant votre jeton de lecture OVHcloud Insight

![grafanaAddConstant](images/grafana2.png){.thumbnail}

6. Cliquez sur `Add`{.action} pour finaliser

7. Cliquez sur l'icône Grafana en haut à gauche et choisissez dans le menu : `Dashboard`{.action}, puis `Import`{.action}

8. Uploader le template suivant : connectivityIspAccess.json TODO

9. Cliquez sur `Import`{.action} pour finaliser


> [!primary]
>
> Félicitations ! Vous avez maintenant votre premier tableau de bord Grafana avec les informations suivantes pour vos accès Internet :
> * Traffic entrant
> * Traffic sortant
>
> Et si votre modem est compatible :
> * Synchronisation, SNR *(Signal to Noise Ratio)*, Atténuation et erreurs CRC *(Contrôle de Redondance Cyclique)*
> * Uptime de la synchronisation, de la connection et du modem
>
