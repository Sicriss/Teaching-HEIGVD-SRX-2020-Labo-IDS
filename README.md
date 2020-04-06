# Teaching-HEIGVD-SRX-2020-Laboratoire-IDS

_par Thoeny Laurent (@Sicriss) & Wojciechowski Cassandre (@CassandreWoj)_



**Ce travail de laboratoire est à faire en équipes de 2 personnes** (oui... en remote...). Je vous laisse vous débrouiller ;-)

**ATTENTION : Commencez par créer un Fork de ce repo et travaillez sur votre fork.**

Clonez le repo sur votre machine. Vous pouvez répondre aux questions en modifiant directement votre clone du README.md ou avec un fichier pdf que vous pourrez uploader sur votre fork.

**Le rendu consiste simplement à répondre à toutes les questions clairement identifiées dans le text avec la mention "Question" et à les accompagner avec des captures. Le rendu doit se faire par une "pull request". Envoyer également le hash du dernier commit et votre username GitHub par email au professeur et à l'assistant**

## Table de matières

[Introduction](#introduction)

[Echéance](#echéance)

[Configuration du réseau](#configuration-du-réseau-sur-virtualbox)

[Installation de Snort](#installation-de-snort-sur-linux)

[Essayer Snort](#essayer-snort)

[Utilisation comme IDS](#utilisation-comme-un-ids)

[Ecriture de règles](#ecriture-de-règles)

[Travail à effectuer](#exercises)


## Echéance 

Ce travail devra être rendu le dimanche après la fin de la 2ème séance de laboratoire, soit au plus tard, **le 6 avril 2020, à 23h59.**


## Introduction

Dans ce travail de laboratoire, vous allez explorer un système de detection contre les intrusions (IDS) dont l'utilisation es très répandue grace au fait qu'il est gratuit et open source. Il s'appelle [Snort](https://www.snort.org). Il existe des versions de Snort pour Linux et pour Windows.

### Les systèmes de detection d'intrusion

Un IDS peut "écouter" tout le traffic de la partie du réseau où il est installé. Sur la base d'une liste de règles, il déclenche des actions sur des paquets qui correspondent à la description de la règle.

Un exemple de règle pourrait être, en language commun : "donner une alerte pour tous les paquets envoyés par le port http à un serveur web dans le réseau, qui contiennent le string 'cmd.exe'". En on peut trouver des règles très similaires dans les règles par défaut de Snort. Elles permettent de détecter, par exemple, si un attaquant essaie d'executer un shell de commandes sur un serveur Web tournant sur Windows. On verra plus tard à quoi ressemblent ces règles.

Snort est un IDS très puissant. Il est gratuit pour l'utilisation personnelle et en entreprise, où il est très utilisé aussi pour la simple raison qu'il est l'un des plus efficaces systèmes IDS.

Snort peut être exécuté comme un logiciel indépendant sur une machine ou comme un service qui tourne après chaque démarrage. Si vous voulez qu'il protège votre réseau, fonctionnant comme un IPS, il faudra l'installer "in-line" avec votre connexion Internet. 

Par exemple, pour une petite entreprise avec un accès Internet avec un modem simple et un switch interconnectant une dizaine d'ordinateurs de bureau, il faudra utiliser une nouvelle machine executant Snort et placée entre le modem et le switch. 


## Matériel

Vous avez besoin de votre ordinateur avec VirtualBox et une VM Kali Linux. Vous trouverez un fichier OVA pour la dernière version de Kali sur `//eistore1/cours/iict/Laboratoires/SRX/Kali` si vous en avez besoin.


## Configuration du réseau sur VirtualBox

Votre VM fonctionnera comme IDS pour "protéger" votre machine hôte (par exemple, si vous faites tourner VirtualBox sur une machine Windows, Snort sera utilisé pour capturer le trafic de Windows vers l'Internet).

Pour cela, il faudra configurer une réseau de la VM en mode "bridge" et activer l'option "Promiscuous Mode" dans les paramètres avancés de l'interface. Le mode bridge dans l'école ne vous permet pas d'accéder à l'Internet depuis votre VM. Vous pouvez donc rajouter une deuxième interface réseau à votre Kali configurée comme NAT. La connexion Internet est indispensable pour installer Snort mais pas vraiment nécessaire pour les manipulations du travail pratique.

Pour les captures avec Snort, assurez-vous de toujours indiquer la bonne interface dans la ligne de commandes, donc, l'interface configurée en mode promiscuous.

![Topologie du réseau virtualisé](images/Snort_Kali.png)


## Installation de Snort sur Linux

On va installer Snort sur Kali Linux. Si vous avez déjà une VM Kali, vous pouvez l'utiliser. Sinon, vous avez la possibilité de copier celle sur `eistore`.

La manière la plus simple c'est de d'installer Snort en ligne de commandes. Il suffit d'utiliser la commande suivante :

```
sudo apt update && apt install snort
```

Ceci télécharge et installe la version la plus récente de Snort.

Vers la fin de l'installation, on vous demande de fournir l'adresse de votre réseau HOME. Il s'agit du réseau que vous voulez protéger. Cela sert à configurer certaines variables pour Snort. Pour les manipulations de ce laboratoire, vous pouvez donner n'importe quelle adresse comme réponse.


## Essayer Snort

Une fois installé, vous pouvez lancer Snort comme un simple "sniffer". Pourtant, ceci capture tous les paquets, ce qui peut produire des fichiers de capture énormes si vous demandez de les journaliser. Il est beaucoup plus efficace d'utiliser des règles pour définir quel type de trafic est intéressant et laisser Snort ignorer le reste.

Snort se comporte de différentes manières en fonction des options que vous passez en ligne de commande au démarrage. Vous pouvez voir la grande liste d'options avec la commande suivante :

```
snort --help
```

On va commencer par observer tout simplement les entêtes des paquets IP utilisant la commande :

```
snort -v -i eth0
```

**ATTENTION : assurez-vous de bien choisir l'interface qui se trouve en mode bridge/promiscuous. Elle n'est peut-être pas eth0 chez-vous!**

Snort s'execute donc et montre sur l'écran tous les entêtes des paquets IP qui traversent l'interface eth0. Cette interface est connectée à l'interface réseau de votre machine hôte à travers le bridge de VirtualBox.

Pour arrêter Snort, il suffit d'utiliser `CTRL-C` (**attention** : il peut arriver de temps à autres que snort ne réponde pas correctement au signal d'arrêt. Dans ce cas-là, il faudra utiliser `kill` pour arrêter le process).

## Utilisation comme un IDS

Pour enregistrer seulement les alertes et pas tout le trafic, on execute Snort en mode IDS. Il faudra donc spécifier un fichier contenant des règles. 

Il faut noter que `/etc/snort/snort.config` contient déjà des références aux fichiers de règles disponibles avec l'installation par défaut. Si on veut tester Snort avec des règles simples, on peut créer un fichier de config personnalisé (par exemple `mysnort.conf`) et importer un seul fichier de règles utilisant la directive "include".

Les fichiers de règles sont normalement stockes dans le repertoire `/etc/snort/rules/`, mais en fait un fichier de config et les fichiers de règles peuvent se trouver dans n'importe quel repertoire. 

Par exemple, créez un fichier de config `mysnort.conf` dans le repertoire `/etc/snort` avec le contenu suivant :

```
include /etc/snort/rules/icmp2.rules
```

Ensuite, créez le fichier de règles `icmp2.rules` dans le repertoire `/etc/snort/rules/` et rajoutez dans ce fichier le contenu suivant :

`alert icmp any any -> any any (msg:"ICMP Packet"; sid:4000001; rev:3;)`

On peut maintenant executer la commande :

```
snort -c /etc/snort/mysnort.conf
```

Vous pouvez maintenant faire quelques pings depuis votre hôte et regarder les résultas dans le fichier d'alertes contenu dans le repertoire `/var/log/snort/`. 


## Ecriture de règles

Snort permet l'écriture de règles qui décrivent des tentatives de exploitation de vulnérabilités bien connues. Les règles Snort prennent en charge à la fois, l'analyse de protocoles et la recherche et identification de contenu.

Il y a deux principes de base à respecter :

* Une règle doit être entièrement contenue dans une seule ligne
* Les règles sont divisées en deux sections logiques : (1) l'entête et (2) les options.

L'entête de la règle contient l'action de la règle, le protocole, les adresses source et destination, et les ports source et destination.

L'option contient des messages d'alerte et de l'information concernant les parties du paquet dont le contenu doit être analysé. Par exemple:

```
alert tcp any any -> 192.168.1.0/24 111 (content:"|00 01 86 a5|"; msg: "mountd access";)
```

Cette règle décrit une alerte générée quand Snort trouve un paquet avec tous les attributs suivants :

* C'est un paquet TCP
* Emis depuis n'importe quelle adresse et depuis n'importe quel port
* A destination du réseau identifié par l'adresse 192.168.1.0/24 sur le port 111

Le text jusqu'au premier parenthèse est l'entête de la règle. 

```
alert tcp any any -> 192.168.1.0/24 111
```

Les parties entre parenthèses sont les options de la règle:

```
(content:"|00 01 86 a5|"; msg: "mountd access";)
```

Les options peuvent apparaître une ou plusieurs fois. Par exemple :

```
alert tcp any any -> any 21 (content:"site exec"; content:"%"; msg:"site
exec buffer overflow attempt";)
```

La clé "content" apparait deux fois parce que les deux strings qui doivent être détectés n'apparaissent pas concaténés dans le paquet mais à des endroits différents. Pour que la règle soit déclenchée, il faut que le paquet contienne **les deux strings** "site exec" et "%". 

Les éléments dans les options d'une règle sont traitées comme un AND logique. La liste complète de règles sont traitées comme une succession de OR.

## Informations de base pour le règles

### Actions :

```
alert tcp any any -> any any (msg:"My Name!"; content:"Skon"; sid:1000001; rev:1;)
```

L'entête contient l'information qui décrit le "qui", le "où" et le "quoi" du paquet. Ça décrit aussi ce qui doit arriver quand un paquet correspond à tous les contenus dans la règle.

Le premier champ dans le règle c'est l'action. L'action dit à Snort ce qui doit être fait quand il trouve un paquet qui correspond à la règle. Il y a six actions :

* alert - générer une alerte et écrire le paquet dans le journal
* log - écrire le paquet dans le journal
* pass - ignorer le paquet
* drop - bloquer le paquet et l'ajouter au journal
* reject - bloquer le paquet, l'ajouter au journal et envoyer un `TCP reset` si le protocole est TCP ou un `ICMP port unreachable` si le protocole est UDP
* sdrop - bloquer le paquet sans écriture dans le journal

### Protocoles :

Le champ suivant c'est le protocole. Il y a trois protocoles IP qui peuvent être analysez par Snort : TCP, UDP et ICMP.


### Adresses IP :

La section suivante traite les adresses IP et les numéros de port. Le mot `any` peut être utilisé pour définir "n'import quelle adresse". On peut utiliser l'adresse d'une seule machine ou un block avec la notation CIDR. 

Un opérateur de négation peut être appliqué aux adresses IP. Cet opérateur indique à Snort d'identifier toutes les adresses IP sauf celle indiquée. L'opérateur de négation est le `!`.

Par exemple, la règle du premier exemple peut être modifiée pour alerter pour le trafic dont l'origine est à l'extérieur du réseau :

```
alert tcp !192.168.1.0/24 any -> 192.168.1.0/24 111
(content: "|00 01 86 a5|"; msg: "external mountd access";)
```

### Numéros de Port :

Les ports peuvent être spécifiés de différentes manières, y-compris `any`, une définition numérique unique, une plage de ports ou une négation.

Les plages de ports utilisent l'opérateur `:`, qui peut être utilisé de différentes manières aussi :

```
log udp any any -> 192.168.1.0/24 1:1024
```

Journaliser le traffic UDP venant d'un port compris entre 1 et 1024.

--

```
log tcp any any -> 192.168.1.0/24 :6000
```

Journaliser le traffic TCP venant d'un port plus bas ou égal à 6000.

--

```
log tcp any :1024 -> 192.168.1.0/24 500:
```

Journaliser le traffic TCP venant d'un port privilégié (bien connu) plus grand ou égal à 500 mais jusqu'au port 1024.


### Opérateur de direction

L'opérateur de direction `->`indique l'orientation ou la "direction" du trafique. 

Il y a aussi un opérateur bidirectionnel, indiqué avec le symbole `<>`, utile pour analyser les deux côtés de la conversation. Par exemple un échange telnet :

```
log 192.168.1.0/24 any <> 192.168.1.0/24 23
```

## Alertes et logs Snort

Si Snort détecte un paquet qui correspond à une règle, il envoie un message d'alerte ou il journalise le message. Les alertes peuvent être envoyées au syslog, journalisées dans un fichier text d'alertes ou affichées directement à l'écran.

Le système envoie **les alertes vers le syslog** et il peut en option envoyer **les paquets "offensifs" vers une structure de repertoires**.

Les alertes sont journalisées via syslog dans le fichier `/var/log/snort/alerts`. Toute alerte se trouvant dans ce fichier aura son paquet correspondant dans le même repertoire, mais sous le fichier `snort.log.xxxxxxxxxx` où `xxxxxxxxxx` est l'heure Unix du commencement du journal.

Avec la règle suivante :

```
alert tcp any any -> 192.168.1.0/24 111
(content:"|00 01 86 a5|"; msg: "mountd access";)
```

un message d'alerte est envoyé à syslog avec l'information "mountd access". Ce message est enregistré dans `/var/log/snort/alerts` et le vrai paquet responsable de l'alerte se trouvera dans un fichier dont le nom sera `/var/log/snort/snort.log.xxxxxxxxxx`.

Les fichiers log sont des fichiers binaires enregistrés en format pcap. Vous pouvez les ouvrir avec Wireshark ou les diriger directement sur la console avec la commande suivante :

```
tcpdump -r /var/log/snort/snort.log.xxxxxxxxxx
```

Vous pouvez aussi utiliser des captures Wireshark ou des fichiers snort.log.xxxxxxxxx comme source d'analyse por Snort.

## Exercices

**Réaliser des captures d'écran des exercices suivants et les ajouter à vos réponses.**

### Essayer de répondre à ces questions en quelques mots :

**Question 1: Qu'est ce que signifie les "preprocesseurs" dans le contexte de Snort ?**

---

**Reponse :**  Il s'agit de modules qui sont exécutés avant l'appel du moteur de règles et qui permettent d'effectuer des opérations sur les paquets avant leur traitement par ce dernier. Cela permet par exemple à certains paquets d'être convertis en texte clair afin de permettre d'y appliquer les règles. [Source](http://manual-snort-org.s3-website-us-east-1.amazonaws.com/node17.html).

_(On imagine qu'aucune capture d'écran n'est nécessaire pour cette question)_.

---

**Question 2: Pourquoi êtes vous confronté au WARNING suivant `"No preprocessors configured for policy 0"` lorsque vous exécutez la commande `snort` avec un fichier de règles ou de configuration "home-made" ?**

---

**Reponse :**  Nous n'avons pas rencontré cette erreur lors du lancement de nos règles personnalisées. Cependant, il semble que ce message indique simplement que le fichier de configuration ne spécifie pas de pré-processeur pour le traitement du flux réseau.

---

--

### Trouver votre nom :

Considérer la règle simple suivante:

alert tcp any any -> any any (msg:"Mon nom!"; content:"Rubinstein"; sid:4000015; rev:1;)

**Question 3: Qu'est-ce qu'elle fait la règle et comment ça fonctionne ?**

---

**Reponse :**  Cette règle cherche dans tous les paquets tcp le nom "Rubinstein" dans le contenu et affiche dans son alerte le message spécifié "Mon nom!".

---

Utiliser un éditeur et créer un fichier `myrules.rules` sur votre répertoire home. Rajouter une règle comme celle montrée avant mais avec votre nom ou un mot clé de votre préférence. Lancer snort avec la commande suivante :

```
sudo snort -c myrules.rules -i eth0
```

**Question 4: Que voyez-vous quand le logiciel est lancé ? Qu'est-ce que tous les messages affichés veulent dire ?**

---

**Reponse :**  Au lancement, Snort fait un résumé de la configuration avec laquelle il est lancé, nous indiquant par exemple qu'une règle en TCP ANY est configurée. Ce que nous voyons est simplement l'initialisation du logiciel mais on ne reçoit pas d'alertes car aucun ne répond aux critères.

![](images/1.png)

![](images/2.png)

---

---

Aller à un site web contenant dans son text votre nom ou votre mot clé que vous avez choisi (il faudra chercher un peu pour trouver un site en http...).

**Question 5: Que voyez-vous sur votre terminal quand vous visitez le site ?**

---

**Reponse :**  On ne constate rien dans le terminal directement, par contre on constate deux choses. Premièrement lors de l'arrêt de Snort on constate qu'il a bien envoyé des alertes. 

![](images/4.png)

Ensuite en allant dans `/var/log/snort/` nous pouvons constater que des alertes sont apparues. Ci-dessous un aperçu d'un `cat alert`.

![](images/5.png)

_(Ici nous avons créé la règle avec le nom "cern" et visité "info.cern.ch" qui est un ancien site HTTP. Nous avons fait cela pour ne pas perdre 2 jours à trouver un site http contenant le nom "Wojciechowski")._

---

Arrêter Snort avec `CTRL-C`.

**Question 6: Que voyez-vous quand vous arrêtez snort ? Décrivez en détail toutes les informations qu'il vous fournit.**

---

**Reponse :**  Snort nous donne des informations sur le type de paquets traités ainsi que sur les actions qu'il a entreprit (par.ex. le nombre d'alertes dans notre cas.)

![](images/6.png)

![](images/7.png)

---


Aller au répertoire /var/log/snort. Ouvrir le fichier `alert`. Vérifier qu'il y ait des alertes pour votre nom ou mot choisi.

**Question 7: A quoi ressemble l'alerte ? Qu'est-ce que chaque élément de l'alerte veut dire ? Décrivez-la en détail !**

---

**Reponse :**  Nous constatons dans l'alerte, tout d'abord le SID ainsi que le numéro de révision, le message que nous avons choisi de lui donner ainsi qu'en dessous la priorité.

Ensuite nous constatons le timestamp de l'alerte, ainsi les adresses IP d'origine et de destination.

Puis en dessous, des informations plus techniques sur le paquet réseau (principalement header) qui ne sont ici pas pertinentes pour la compréhension.

![](images/8.png)

---


--

### Detecter une visite à Wikipedia

Ecrire une règle qui journalise (sans alerter) un message à chaque fois que Wikipedia est visité **DEPUIS VOTRE** station. **Ne pas utiliser une règle qui détecte un string ou du contenu**.

**Question 8: Quelle est votre règle ? Où le message a-t'il été journalisé ? Qu'est-ce qui a été journalisé ?**

---

**Reponse :**  Notre règle est la suivante :

`log tcp 192.168.43.107 any -> 91.198.174.192 any (sid 4000025; rev:1;)`

Nous imaginons que de détecter "Wikipedia.org" dans le contenu engendrerait un grand nombre de faux positif étant donné le nombre de fois où le site est référencé, mais la solution d'utiliser l'adresse IP n'est pas idéale.

Les messages sont journalisés dans le fichier log de la capture qui est généré dans `/var/log/snort/` à nouveau. Ci-dessous se trouve une capture du tcpdump de ce log. On constate que de nombreuses requêtes sont journalisées puisque l'accès à la page ne consiste pas en un seul paquet.

_(ici on utilise any comme port mais on pourrait utiliser 443, 80, ou les deux, afin de détecter spécifiquement de l'HTTP/HTTPS)_.

![](images/9.png)

---

--

### Detecter un ping d'un autre système

Ecrire une règle qui alerte à chaque fois que votre système reçoit un ping depuis une autre machine (je sais que la situation actuelle du Covid-19 ne vous permet pas de vous mettre ensemble... utilisez votre imagination pour trouver la solution à cette question !). Assurez-vous que **ça n'alerte pas** quand c'est vous qui envoyez le ping vers un autre système !

**Question 9: Quelle est votre règle ?**

---

**Reponse :**  Notre règle est ci-dessous

`alert icmp any any -> 192.168.43.107 any (msg:"has been pinged"; itype:8; sid:4000027; rev:1;)`

---

**Question 10: Comment avez-vous fait pour que ça identifie seulement les pings entrants ?**

---

**Reponse :**  Le fait d'utiliser le symbole `->` permet que la règle soit à sens unique. Ci-dessous on verra comment rendre la règle bi-directionnelle.

Nous avons dû ajouter `itype:8` afin de vérifier qu'on ne détecte pas les réponses à des ping sortants dans cette règle. Une alternative aurait probablement été de détecter "ICMP Request" dans le content.

---


**Question 11: Où le message a-t-il été journalisé ?**

---

**Reponse :**  Le message a été journalisé dans le fichier de log correspondant à la capture, en plus d'apparaître dans les alertes.

---

**Question 12: Qu'est-ce qui a été journalisé ?**

---

**Reponse :**  Ci-dessous un extrait du log correspondant (le ping a été manuellement arrêté après 3).

![](images/10.png)

---

--

### Detecter les ping dans les deux sens

Modifier votre règle pour que les pings soient détectés dans les deux sens.

**Question 13: Qu'est-ce que vous avez modifié pour que la règle détecte maintenant le trafic dans les deux senses ?**

---

**Reponse :**  Simplement nous avons converti notre flèche en `<>` afin que la règle soit bi-directionnelle.

---

![](images/11.png)

--

### Detecter une tentative de login SSH

Essayer d'écrire une règle qui Alerte qu'une tentative de session SSH a été faite depuis la machine d'un voisin (je sais que la situation actuelle du Covid-19 ne vous permet pas de vous mettre ensemble... utilisez votre imagination pour trouver la solution à cette question !). Si vous avez besoin de plus d'information sur ce qui décrit cette tentative (adresses, ports, protocoles), servez-vous de Wireshark pour analyser les échanges lors de la requête de connexion depuis votre voisin.

**Question 14: Quelle est votre règle ? Montrer la règle et expliquer en détail comment elle fonctionne.**

---

**Reponse :**  `alert tcp any any -> any 22 (content:"SSH-"; sid:4000029; rev:1;)`

La règle détecte tout d'abord les paquets tcp en destination du port 22, puis vérifie dans le contenu qu'il y ait "SSH-" (pour détecter par exemple SSH-2.0). Cela nous permet d'éviter les faux-positifs par exemple si un scan venait à contacter le port 22.

---


**Question 15: Montrer le message d'alerte enregistré dans le fichier d'alertes.** 

---

**Reponse :**  

![](images/12.png)

---

--

### Analyse de logs

Lancer Wireshark et faire une capture du trafic sur l'interface connectée au bridge. Générez du trafic avec votre machine hôte qui corresponde à l'une des règles que vous avez ajouté à votre fichier de configuration personnel. Arrêtez la capture et enregistrez-la dans un fichier.

**Question 16: Quelle est l'option de Snort qui permet d'analyser un fichier pcap ou un fichier log ?**

---

**Reponse :**  On peut utiliser snort `-r` fichier.pcap afin de lire depuis une capture Wireshark. Bien entendu il faut également préciser `-c configfile` si on veut y appliquer nos règles personnelles.

---

Utiliser l'option correcte de Snort pour analyser le fichier de capture Wireshark.

**Question 17: Quelle est le comportement de Snort avec un fichier de capture ? Y-a-t'il une difference par rapport à l'analyse en temps réel ?**

---

**Reponse :**  Les alertes sont affichées lors de l'exécution mais rien n'est enregistré. Une autre différence est que le résumé final (nombre d'alertes et de log générés lorsque Snort s'arrête) n'est pas disponible.

![](images/13.png)

![](images/14.png)

---

**Question 18: Est-ce que des alertes sont aussi enregistrées dans le fichier d'alertes?**

---

**Reponse :**  Les nouvelles alertes ne sont pas ajoutée dans `/var/log/snort/alert` et aucun nouveau fichier de log n'est créé. 

---

--

### Contournement de la détection

Faire des recherches à propos des outils `fragroute` et `fragtest`.

**Question 20: A quoi servent ces deux outils ?**

---

**Reponse :**  Ces outils servent à modifier le trafic réseau. Ici ils permettent d'échapper à la détéction de l'IDS. Fragtest fait partie de fragroute et sert tester si des paquets sont correctement transmis ou s'ils sont drop (par.ex. par un firewall).

**Question 21: Quel est le principe de fonctionnement ?**

---

**Reponse :**  Ces outils permettent la ré-écriture de trafic réseau à destination d'un hôte, permettant dans le cas présent de modifier les paquets transmis (par exemple les ré-organise).

---


**Question 22: Qu'est-ce que le `Frag3 Preprocessor` ? A quoi ça sert et comment ça fonctionne ?**

---

**Reponse :**  C'est l'un des modules pré-processeur de Snort (réponse à une question plus haut pour d'avantage de détails). Il fait principalement de la "défragmentation" dans le but d'améliorer le temps d'exécution du traitement des données, mais permet également (ce qui nous intéresse ici) des techniques "anti-evasion". [Source](https://www.snort.org/faq/readme-frag3)

---


Reprendre l'exercice de la partie [Trouver votre nom](#trouver-votre-nom-). Essayer d'offusquer la détection avec `fragroute`.

**Question 23: Quel est le résultat de votre tentative ?**

---

**Reponse :**  Nous avons executé `fragroute 192.168.43.234` en laissant `/etc/fragroute.conf` à ses valeurs par défaut, malheureusement impossible de corriger l'erreur rencontrée, le détail de l'erreur rencontrée est disponible en fin de ce rapport.

Selon notre compréhension de l'outil, nous ne serons pas detecté en utilisant la commande ainsi car fragroute va offusquer l'information.

---


Modifier le fichier `myrules.rules` pour que snort utiliser le `Frag3 Preprocessor` et refaire la tentative.

**Question 24: Quel est le résultat ?**

---

**Reponse :**  

Comme ci-dessus, nous sommes bloqués par l'erreur rencontrée. Cependant, nous imaginons que le preprocesseur "frag3" est une réponse directe suffisamment efficace à fragroute pour permettre la détéction.

---


**Question 25: A quoi sert le `SSL/TLS Preprocessor` ?**

---

**Reponse :**  Le but de ce préprocesseur est de determiner si le trafic est chiffré et de permettre à ce dernier d'être (en tout cas partiellement) ignoré. Le trafic chiffré affecte les performances et va tendre à produire d'avantage de faux-positifs.

Selon la documentation, par exemple du trafic HTTPS ne sera vérifié qu'au niveau du handshake.

---

**Question 26: A quoi sert le `Sensitive Data Preprocessor` ?**

---

**Reponse :**  Ce module Snort détecte les informations personnelles (Numéro de sécurité sociale, e-mail, numéro de carte bancaire, etc ..).

---

### Conclusions


**Question 27: Donnez-nous vos conclusions et votre opinion à propos de snort**

---

**Reponse :**  Snort nous permet être un très bon outil pour le rôle de NIDS, en particulier les modules / pré-processeurs dont nous avons étudiés la documentation semblent très puissants, son utilisation dans ce laboratoire nous a également permis de raffraichir la théorie des vidéos sur le sujet.

Il s'agit cependant d'un outil plus complexe à configurer, même pour des règles simples comme les notres nous avons dû appliquer des corrections pour améliorer la détéction (p.ex. question 14) et il est donc plus difficile à prendre en main.

---



##### Annexe : Description de notre erreur fragroute

Notre erreur rencontrée sur la machine Kali avec fragroute consistait à un message d'erreur `fragroute : no route to 192.168.43.107 : No such process`. L'adresse specifiée était l'adresse locale mais nous avons également testé la machine hôte et d'autres adresses, sans succès. L'erreur laisse penser à un problème réseau mais les autres communications (ping, ssh, etc..) fonctionnaient correctement entre les machines.

![](images/fragroute0.png)

Nous avons fait des recherches concernant cette erreur, sans succès, puis contacté l'assistant mais sans succès également (merci Yann pour ton temps !).

Nous souhaitions essayer sur un autre environnement que celui de la machine Kali, ainsi nous avons voulu build snort et fragroute sur un linux natif (arch-based) mais avec une erreur lors du _make_ de fragroute, nous nous sommes retrouvés bloqués également.

![](images/fragroute.png)



<sub>This guide draws heavily on http://cs.mvnu.edu/twiki/bin/view/Main/CisLab82014</sub>