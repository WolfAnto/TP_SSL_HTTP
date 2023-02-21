# VM Autorité de certification

Les paquets à installer :
```bash
apt install apache2
apt install openssl
```

Allez dans le répoiture de configuration de votre PKI
```bash
cd /etc/ssl
```

Création de la clé privée de l'autorité de certification :
```bash
openssl genrsa -des3 2048 > ca.key
```
A partir de la clé privée, créé un certificat x509
```bash
nano ca.conf
```
```html
[ req ]
default_bits = 2048
encrypt_key = yes
distinguished_name = req_dn
prompt = no
[ req_dn ]
C=FR
ST=France
L=LRY
O=iut
OU=r&t
CN=CA_rt
emailAddress=admin@rt.iut
```
```bash
openssl req -config ca.conf -new -x509 -sha256 -days 365 -key ca.key > ca.crt
```
Saisir la passphrase
```bash
Exemple :
rtlry
```

# VM Serveur WEB

Générer la clef privée en définissant un nom de fichier :
```bash
openssl genrsa 2048 > server.key
```
Observer son contenu
```bash
less server.key
```
Protégez votre fichier en faisant :
```bash
chmod 400 server.key
```
Créer un fichier de demande de signature de certificat
```bash
nano server.conf
```
```bash
[ req ]
default_bits = 2048
encrypt_key = yes
distinguished_name = req_dn
x509_extensions = cert_type
prompt = no
[ req_dn ]
C=FR
ST=France
L=LRY
O=iut
OU=r&t
CN=www.rt.iut
emailAddress=admin@rt.iut
[ cert_type ]
nsCertType = server
```
Créer le formulaire de demande de certificat (fichier serveur.csr) à partir de notre clé privé 
```bash
openssl req -config server.conf -new -sha256 -key server.key > server.csr
```
Envoyer le fichier server.csr à l'autorité de certification
```bash
(nécéssite SSH sur les deux machines)
scp server.csr rt@191.162.X.X:/home/rt/Bureau/server.csr
```

# VM Autorité de certification

Récupérer le fichier server.csr
```bash
cp /home/rt/Desktop/server.csr /etc/ssl/
```
Créer un fichier de certification version 3
```bash
nano v3.ext
```
```bash
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = www.rt.iut
DNS.2 = localhost
```
Signer la demande de cerficat
```bash
openssl x509 -req -sha256 -extfile v3.ext -in server.csr -out server.crt -CA ca.crt -CAkey ca.key -CAcreateserial -CAserial ca.srl
```
Afficher le certificat
```bash
less server.crt
```
Envoyer le fichier server.csr à l'autorité de certification
```bash
scp server.crt rt@191.162.X.X:/home/rt/Bureau/server.crt
```

# VM Serveur WEB

Récupérer le fichier server.csr
```bash
cp /home/rt/Desktop/server.crt /etc/ssl/certs/server.crt
```
Editez le fichier de configuration default-ssl
```bash
nano /etc/apache2/site-available/default-ssl
```
Vérifier que le serveur écoute bien sur le port 443 et qu'il utilise bien les bons certificats et clé.
```html
<VirtualHost _DEFAULT_:443>
 ServerName localhost:443
 DocumentRoot /var/www/html
SSLProtocol all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
#On installe les certificats et clé pour ce serveur Web
# Server Certificate:
SSLCertificateFile /etc/ssl/certs/server.crt
# Server Private Key :
SSLCertificateKeyFile /etc/ssl/private/server.key
# Server Certificate Chain CA :
SSLCertificateChainFile /etc/ssl/certs/ca.crt
</VirtualHost>
```
Activer le module ssl
```bash
a2enmod ssl
```
Activer le virtualhost ssl
```bash
a2ensite default-ssl
```
Relancez le serveur
```bash
systemctl restart apache2
```

# Client HTTPS

À partir d'un navigateur http sur votre poste serveur :
```bash
https://localhost
```
Récupérer le fichier ca.crt du serveur Autorité de certification
```bash
(Si le client est sous linux)
(Depuis le serveur Autorité de certification vers Client HTTPS)
scp ca.crt rt@191.162.X.X:/home/rt/Bureau/server.crt
```
```bash
(Si le client est sous Windows)
(Se connecter avec WinSCP sur le serveur Autorité de certification)
```
Sous Windows, un clic droit sur le fichier devrait vous permettre de lancer l'assistant d'installation du certificat dans Internet Explorer. Vous devez l'installer en tant qu'autorité de certification.Vérifiez en suite que le certificat s'y trouve bien sous le nom de "cert_CA".

Sur Mozilla Firefox :
```bash
Edition > Préférence > Confidentialité et Sécurité > Certificats
```
```bash
Bouton de commande : gestion des certificats
Onglet : autorité
Bouton de commande : importer
etc...
```
```bash

```
```bash

```
