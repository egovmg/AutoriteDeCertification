# AutoriteDeCertification
# Installation d’un PKI <a href="https://www.ejbca.org/">EJBCA </a>

Ces sont les instructions pour l’installation d’un PKI, l’Autorité des certificats et ses services associés, à l’aide du logiciel gratuit EJBCA. Les instructions peuvent être suivies à l’aide d’Ubuntu Server 18.04 LTS . L’EJBCA est la mise en œuvre de référence d’un PKI qui offre un grand nombre de services associés à la signature électronique.

# Préréquis

OpenJDK 8

<a href="https://developers.redhat.com/products/eap/overview">JBoss EAP 7.0</a>

<a href="https://sourceforge.net/projects/ejbca/files/ejbca6/ejbca_6_10_1_2/">EJBCA CE 6.10</a>

# Instructions

1- Téléchargez ce référentiel et enregistrez-le sur votre serveur à l’intérieur du dossier /opt. Assurez-vous de télécharger et d’ajouter le serveur JBoss EAP 7 et EJBCA CE 6.15 sur ce même dossier (jboss-eap-7.0.0.zip ,  ejbca_ce_6_10_1_2.zip et installation.sh )

2- Préparez votre serveur en exécutant vos commandes système d’exploitation ( installation.sh).
  
  $cd AutoriteDeCertification
  
  AutoriteDeCertification/$ chmod +x installation.sh
 
 AutoriteDeCertification/$ ./installation.sh

3- Modifier la variable 'ca.dn' dans /opt/ejbca/conf/install.properties. Il s’agit du nom de l’autorité de certification administrative créée par défaut et n’a aucun lien avec le certificat racine que nous créerons ultérieurement.

4- Modifier le fichier /opt/ejbca/conf/web.properties, n’oubliez pas de prendre note des informations d’identification Utilisateur Super Admin : superadmin.cn, superadmin.dn, superadmin.password.

5- Modifier le fichier /opt/ejbca/conf/web.properties, n’oubliez pas de prendre note du nom de domaine et des informations d’identification de configuration HTTPS : httpsserver.password, httpsserver.dn, httpsserver.hostname, java.trustpassword.

6- Entrez le dossier '/opt/scripts' et modifiez le fichier conf-jboss05.cli, assurez-vous d’utiliser les valeurs de l’étape précédente .

7- Entrez le dossier '/opt/scripts', ouvrez le fichier commandes-jboss.txt. Exécutez ces commandes une par une, vous devez vous assurer que JBoss traite chaque commande avec succès une par une. À la fin assurez-vous de copier et d’installer le certificat superadmin sur la machine que vous voulez (ce certificat se trouve dans ejbca/p12/superadmin.p12).

8- L’interface de publication sera disponible à l’adresse http://[ip server]:8080/ejbca et http://[ip server]:4447/ejbca.

9- Pour les machines dont le certificat ' superadmin' est installé, la page d’administration sera disponible à l’adresse https://[ip server]:8443/ejbca/adminweb/.

# Profils de certificats (Pas encore terminés)

Ces profils définissent les caractéristiques techniques des certificats. En plus des profils que EJBCA installe par défaut, le fichier profiles-cert.zip contient des profils de certificats pouvant être utilisés comme référence. Pour les utiliser dans la page Administration, sélectionnez « Fonctions de l’autorité de certification », « Profils de certificat », « Sélectionner un fichier », « Importer à partir du fichier Zip ». Une fois les certificats importés, utilisez l’option « Autorités de certificats » pour créer un ca racine et deux autorités de certification subordonnées comme indiqué dans le diagramme suivant.

             RootCA
          ------ | --------
          |               |             
      SubCAPersonne   SubCAServices


Consultez enfin les profils que vous avez importés, assurez-vous qu’ils utilisent les CA que vous venez de créer et effectuez les ajustements que vous jugez nécessaires.

# Profils d’entités (pas encore terminés)

Ces profils définissent le contenu des certificats pour les utilisateurs ou les entités finales. Le fichier profiles-enti.zip contient des profils d’entités pouvant être utilisés comme référence. Pour les utiliser dans la page Administration, sélectionnez 'Fonctions RA', 'Profils d’entité de fin', 'Sélectionner le fichier', 'Importer à partir du fichier Zip'.

Enfin, examinez les profils importés, assurez-vous qu’ils utilisent les profils de certificats installés à l’étape précédente et effectuez les ajustements que vous jugez nécessaires. Vous pouvez ajouter des utilisateurs à partir de l’option 'Ajouter une entité de fin'. Chaque entité est un nouvel utilisateur qui pourra se connecter à partir du site Web public http://localhost:8080/ejbca et obtenir son certificat.

# Configuration de service OCSP 

Le service OCSP permet à un utilisateur ou à une application de valider l’état (révoqué, actif, expiré, etc.) d’un certificat auprès de l’Autorité des certificats correspondante en temps réel.

Cette installation utilise l’EJBCA comme autorité de certification et autorité de validation de l’OCSP. Vous pouvez configurer EJBCA pour agir en tant qu’autorité de validation dédiée et offrir uniquement le service OCSP. Voici les étapes suivantes pour ajouter et configurer le service OCSP.

Certificat OCSP : chaque requête ocsp reçoit une réponse signée électroniquement, nous devons donc disposer d’un certificat pour signer les requêtes OCSP (Utilisation de clés étendues). Nous ajouterons une nouvelle entité à l’aide du profil « validation-ocsp », lié à l’autorité de certification « Sous-services ». Cette entité sera utilisée pour générer le certificat qui signe les réponses oCSP, avant de le générer à partir de l’interface publique effectuer les étapes suivantes :

Magasin de clés : nous allons créer un magasin (Jeton Crypto) pour stocker les clés que le service OCSP utilisera. Dans le menu principal, sélectionnez :

'FUNCTIONS CA' -> 'Crypto Token' -> 'Create New '

Donnez-lui un nom, marquez-le comme actif et générez une nouvelle paire d’accolades.

Liaison de clé : dans cette étape, nous associerons le magasin et les accolades de l’étape précédente à un nouveau service OCSP, choisissez : 'System Functions' -> 'Internal Key Bindings' -> OcspKeyBindings -> 'Create new'
Donnez-lui un nom et sélectionnez le jeton crypto créé à l’étape précédente.

Créer une demande de CSR : dans cette étape, nous créerons une demande de signature (CSR) pour utiliser l’entité créée à l’étape 1. Dans la liste des OcspKeyBindings, sélectionnez l’élément créé à l’étape précédente et dans la colonne ' Action' sélectionnez 'CSR'. Ensuite, à partir de la page publique EJBCA (http://localhost:8080/ejbca) sélectionnez « Créer un certificat à partir de la CSR », utilisez les informations d’identification de l’entité créée à l’étape 1 et téléchargez le fichier CSR.

Activer le service OCSP : à partir de ' Liaisons de clés internes' -> 'OcspKeyBindings' sélectionnez l’élément créé à l’étape 3 et dans la colonne 'Action' sélectionnez 'Update'. Le système trouvera le certificat nouvellement généré à l’aide des clés que notre jeton Crypto possède déjà. Sélectionnez enfin le bouton « Activer » pour laisser le service OCSP actif.

Pour tester le service, vous pouvez utiliser OpenSSL et les certificats de l’autorité de certification :

         openssl ocsp -req_text -issuer subCA.pem -CAfile RootCA.pem -cert entite.pem  -url http://localhost:8080/ejbca/publicweb/status/ocsp  
# URL de requête OCSP
Ensuite, nous allons créer une URL de requête OCSP plus conviviale. Pour cela, nous allons utiliser NGINX comme un proxy inverse.

         apt-get install nginx-light

Modifiez les paramètres dans /etc/nginx/sites/sites-available/default, puis ajoutez le bloc suivant :

       location /ocsp {
          proxy_pass http://localhost:8080/ejbca/publicweb/status/ocsp;
           proxy_read_timeout 30s;
        }
Après le redémarrage du service NGINX, nous pourrons interroger le service OCSP à l’aide de la nouvelle URL :

         openssl ocsp -req_text -issuer subCA.pem -CAfile RootCA.pem -cert entite.pem  -url http://localhost/ocsp  

# Service d'Horodatage ou Time Stamp Autority (TSA)

Les mêmes créateurs EJBCA offrent un serveur de signature haute performance qui inclut le service d’horodatage, les instructions d’installation sont disponibles dans ce lien <a href="https://github.com/egovmg/AutoriteDHorodatage-TSA">TSA</a>(A faire).

# Licence
Ce travail est couvert dans le cadre de la stratégie de développement des services de gouvernance électronique du Gouvernement Malagasy et, à ce titre, est un travail de valeur publique soumis aux lignes directrices de la Politique sur les données ouvertes et de la licence CC-BY-SA.

