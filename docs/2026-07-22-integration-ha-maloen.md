# 2026-07-22 Intégration Home Assistant avec Maloen

Ce document décrit les premiers changements apportés pour intégrer la Cofybox avec
les services pré-existants centralisés (commande, EMS...).

Sans la Cofybox, les équipements domotiques se connectent directement au courtier MQTT central.
Avec la Cofybox, les équipements se connectent sur le réseau local à Home Assistant et à un courtier MQTT local.
Les mêmes cas d'utilisation doivent fonctionner dans les deux cas. En plus, le tableau de bord dans Home Assistant doit permettre de faire les mêmes choses que l'écran, à savoir programmer une consommation flexible.

## Option choisie

**Pont MQTT et automatisation Home Assistant** avec partage des sujets dans les deux sens sans renommage et sans gestion de contrôle d'accès.
Ces deux aspects pourront être abordés ultérieurement.

Avantage : rapidité de mise en œuvre
Inconvénient : partage des messages entre toutes les Cofybox

## Options considérées

### Pont MQTT et automatisations Home Assistant

Le courtier MQTT local peut être configuré pour faire un pont avec le MQTT central (mqtt.projet-elfe.fr).
Certains sujets peuvent être partagés dans les deux sens. Par exemple, tous les sujets commençant par `hasp/` peuvent être partagés afin que les messages venant d'un écran (ex : `hasp/g001/state/p6b5`) puissent arriver au courtier central et, inversement, que les messages venant de commande (ex : `hasp/g001/command/jsonl`) puissent arriver à l'écran.

Les sujets suivants seraient partagés :

- `hasp/#` pour les écrans
- `tele/#` pour les prises commandées
- `cmnd/#` idem

_Réplication des messages avec renommage._
Lorsque les sujets sont partagés, on peut aussi configurer un renommage pour changer un préfixe local en un préfixe distant.
Par exemple, les sujets `hasp/#` en local pourraient être mis en correspondance avec les sujets `cofybox/cb001/hasp/#` en central.

_Contrôle d'accès._
Le courtier MQTT central peut être configuré pour que les courtiers des Cofybox ne puissent accéder qu'aux sujets qui les concerne.
Ceci serait d'autant plus facile à configurer si tous les sujets comportent un même préfixe, comme `cofybox/cb001/#`.

### Script Python dédié

Un script Python serait développé pour se connecter aux courtiers local et central et transcrire les messages. Ce script pourrait également intéragir avec l'[API REST](https://developers.home-assistant.io/docs/api/rest/) de Home Assistant.

Le script serait packagé dans une image de conteneur dédiée.
Ce conteneur supplémentaire serait ajouté au Docker Compose existant.

### Intégration Home Assistant

Une intégration serait développé pour faire la même chose que l'option script Python.
Au lieu que cette intégration tourne dans un conteneur Docker dédié, elle serait installée dans Home Assistant directement.
