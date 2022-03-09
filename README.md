# L7-Protection
Un moyen simple de protéger un serveur web d'attaques par déni de service couche 7


# Le Layer 7 en très bref.
Le layer 7 touche principalement aux processus d'une machine. N'étant pas assez expérimenté sur le sujet, je préfère rediriger vers [cette page](https://www.cloudflare.com/fr-fr/learning/ddos/what-is-layer-7/) appartenant à Cloudflare qui explique très bien ce qu'est le layer 7 dans les attaques par déni de service.

# Mon intérêt dans la protection d'attaques L7.
Les attaques L7 sont actuellement les plus faciles à réaliser. Il suffit d'un simple accès à une machine linux. J'utilise des Google Cloud Console disponibles gratuitement via le portail Google Cloud. Il existe une multitude de scripts permettant de pratiquer ces attaques ce qui fait que de nombreuses personnes puissent lancer ce genre d'attaque et ce sans même savoir de quoi il s'agit.
J'aide une association française du nom de Project Heberg hébergeant des applications nodeJS, java et python. Ils ont subit il y a peu de nombreuses attaques par déni de service. J'ai rejoint l'équipe en tant qu'analyste de sécurité afin de trouver des solutions aux problèmes d'attaques ainsi que d'améliorer la sécurité de l'infrastructure de l'hébergeur.

# Analyse d'une requête Layer 7 grâce au serveur web Nginx.
Pour contrer ce type d'attaque, j'ai dû comprendre ce qu'elle envoyait comme requête. J'ai crée un serveur Nginx sous docker via le panel Pterodactyl permettant de visualiser tout cela assez facilement. On peut donc récupérer ce genre d'informations :

![](https://cdn.discordapp.com/attachments/781278473960423484/932272922838573086/unknown.png)

On voit donc que pour ce script ut

NB: L'addresse de requête est l'adresse du routeur (10.8.0.1) car le serveur docker est derrière un tunnel IP. Les IP renvoyées sont donc celles du tunnel agissant comme routeur.

# Bloquer les requêtes Layer7 sur vos serveurs web.
## Solutions via Cloudflare
Pour cette solution, il "suffit" de mettre votre site internet sur la gestion DNS de cloudflare pour activer le proxy permettant à cloudflare de mitiger les attaques. Attention que vous devez changer d'ip car cloudflare ne protègera que votre domaine. Même si votre site n'est pas connu, il est fort probable que des sites comme securitytrails aient encore des logs des IP utilisées pour votre domaine.
ATTENTION ! Cloudflare en faisant un proxy va changer le logging d'ip de votre système. Pour fixer cela, ajoutez les lignes suivantes dans le bloc http de votre configuration nginx :
```
        ##
        # CF IP
        ##
        set_real_ip_from 103.21.244.0/22;
        set_real_ip_from 103.22.200.0/22;
        set_real_ip_from 103.31.4.0/22;
        set_real_ip_from 104.16.0.0/12;
        set_real_ip_from 108.162.192.0/18;
        set_real_ip_from 131.0.72.0/22;
        set_real_ip_from 141.101.64.0/18;
        set_real_ip_from 162.158.0.0/15;
        set_real_ip_from 172.64.0.0/13;
        set_real_ip_from 173.245.48.0/20;
        set_real_ip_from 188.114.96.0/20;
        set_real_ip_from 190.93.240.0/20;
        set_real_ip_from 197.234.240.0/22;
        set_real_ip_from 198.41.128.0/17;
        set_real_ip_from 2400:cb00::/32;
        set_real_ip_from 2606:4700::/32;
        set_real_ip_from 2803:f800::/32;
        set_real_ip_from 2405:b500::/32;
        set_real_ip_from 2405:8100::/32;
        set_real_ip_from 2c0f:f248::/32;
        set_real_ip_from 2a06:98c0::/29;
        real_ip_header CF-Connecting-IP;
```
Ce bloc va ajouter la véritable IP dans les headers renvoyés par cloudflare. Les IP dans la configuration sont les adresses IP de cloudlfare fournies [ici](https://www.cloudflare.com/ips/)


## Solutions via votre serveur web.
### Nginx
Ouvrez votre fichier de configuration situé sous /etc/nginx/nginx.conf
ajoutez dans le bloc http la condition suivante :
```
if ($request_method = HEAD) {
                return 444;
        }
```
Cette condition va bloquer les requêtes HEAD de votre serveur web Nginx. Cela ne pourra pas stopper totalement les attaques car le serveur devra quand même renvoyer l'erreur 444 mais cela diminuera grandement l'attaque car elle ne devra plus traîter de données.

### Apache (non testé)
N'utilisant pas apache2, je ne peux pas certifier que cela fonctionne mais il suffirait d'ajouter dans le .htaccess ce bloc :
```
    <Limit HEAD>
    Order Deny,Allow
    Deny from All
    </Limit>
```

Ces solutions sont évidemment à adapter à vos besoins, si votre site demande l'accès aux requêtes HEAD, essayez plûtot d'ajouter une condition supplémentaire basée sur l'addresse IP. Il doit aussi être possible de faire des choses plus simples avec UFW mais je n'ai pas encore exploré cette piste.
