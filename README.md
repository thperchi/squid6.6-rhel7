# Squid 6.6 Installation sur CentOS 7 avec Docker

Ce projet documente le processus d'installation de Squid 6.6 sur CentOS 7 en utilisant Docker. Il inclut la création d'un conteneur Docker personnalisé, la compilation de Squid à partir du code source, et des suggestions pour créer un package RPM de Squid.

## Structure du Projet

- `Dockerfile` : Fichier pour construire l'image Docker de CentOS 7 avec les dépendances nécessaires.
- `docker-compose.yml` : Fichier pour gérer le conteneur Docker avec des volumes pour le code source et les logs.
- `squid-6.6/` : Répertoire contenant le code source de Squid.

## Installation et Configuration

### Création de l'Image Docker

**Dockerfile** :

```
FROM centos:7

RUN yum -y update && yum -y install \
    wget \
    net-tools \
    vim

WORKDIR /squid-6.6

CMD ["/bin/bash"]
```

**Docker-Compose** :

```
version: '3'
services:
  centos7:
    build: .
    volumes:
      - ./squid-6.6:/squid-6.6
        - ./logs:/logs
    stdin_open: true
    tty: true
```

### Configuration du Conteneur

Après le démarrage du conteneur, exécutez les commandes suivantes :

```
yum -y groupinstall "Development Tools"
yum -y install centos-release-scl
yum -y install devtoolset-8
scl enable devtoolset-8 'bash'
```

### Compilation de Squid

**Configuration** :

```
./configure --prefix=/usr \
  --includedir=/usr/include \
  --datadir=/usr/share \
  --bindir=/usr/sbin \
  --libexecdir=/usr/lib/squid \
  --localstatedir=/var \
  --sysconfdir=/etc/squid
  ```

**Compilation et Installation** :

```
make all
make install
```

## Création d'un Package RPM de Squid 6.6 pour RHEL 7

### Installation de rpmbuild

Avant de commencer, assurez-vous que `rpmbuild` est installé sur votre système RHEL 7. Si ce n'est pas le cas, vous pouvez l'installer en utilisant `yum` :

    yum install rpm-build rpmdevtools

Cela installera `rpmbuild` ainsi que d'autres outils utiles pour le développement de RPM.

### Configuration de l'Environnement de Construction RPM

Après avoir installé `rpmbuild`, configurez votre environnement de construction :

1. **Créez l'Arborescence de Répertoires rpmbuild** :
   Exécutez la commande suivante pour créer l'arborescence nécessaire :
   
    ```
    rpmdev-setuptree
    ```

   Cela créera un dossier `rpmbuild` dans votre répertoire personnel avec plusieurs sous-dossiers : `BUILD`, `RPMS`, `SOURCES`, `SPECS`, et `SRPMS`.

### Préparation du Fichier Spec pour Squid

2. **Créez le Fichier Spec pour Squid** :
   - Ouvrez un éditeur de texte et créez un nouveau fichier. 
   - Enregistrez ce fichier dans `~/rpmbuild/SPECS/` avec un nom approprié, par exemple `squid-6.6.spec`.

   Voici un premier jet d'un fichier spec pour Squid :

```
Name:           squid
Version:        6.6
Release:        1%{?dist}
Summary:        Squid - The Internet Object Cache

License:        GPL
URL:            http://www.squid-cache.org
Source0:        http://www.squid-cache.org/Versions/v6/squid-6.6.tar.gz

BuildRequires:  gcc
BuildRequires:  make
BuildRequires:  libtool
BuildRequires:  openssl-devel

Requires:       openssl

%description
Squid 6.6 rpm for RHEL 7.3

%prep
%setup -q

%build
./configure --prefix=/usr \
  --includedir=/usr/include \
  --datadir=/usr/share \
  --bindir=/usr/sbin \
  --libexecdir=/usr/lib/squid \
  --localstatedir=/var \
  --sysconfdir=/etc/squid
make %{?_smp_mflags}

%install
rm -rf $RPM_BUILD_ROOT
make install DESTDIR=$RPM_BUILD_ROOT

%files
%defattr(-,root,root,-)
/usr/sbin/squid
/etc/squid
/var/spool/squid
/var/log/squid
/usr/lib/squid

```

### Obtenir le Code Source de Squid

3. **Télécharger le Code Source** :
   - Téléchargez le code source de Squid 6.6 (https://src.fedoraproject.org/repo/pkgs/squid/squid-6.6.tar.xz/sha512/4ab261ed85ad674288467500aca9d8a48e3918b55f777635c0ba7a2551f248d35536848a5fbf2c946490a818004727f2aed33144f0a3ebab0be36cc4cffb020c/). Assurez-vous que le fichier source correspond à l'URL spécifiée dans votre fichier spec (`Source0`).
   - Placez le fichier tar.gz téléchargé dans `~/rpmbuild/SOURCES/`.

### Construire le Package RPM

4. **Construire le RPM** :
   - Ouvrez un terminal.
   - Naviguez à votre répertoire `SPECS` :

    ```
    cd ~/rpmbuild/SPECS/
    ```

   - Exécutez la commande suivante pour construire le package RPM :
   
    ```
    rpmbuild -ba squid-6.6.spec
    ```

   - Cette commande va lancer le processus de compilation et de création du package RPM en utilisant les instructions dans le fichier spec.

### Tester le Package RPM

5. **Tester le Package RPM** :
   - Il est conseillé de tester l'installation du package RPM dans un environnement de test avant de l'utiliser en production.
   - Installez le package RPM pour tester son installation et sa fonctionnalité :
   
    ```
    yum localinstall ~/rpmbuild/RPMS/x86_64/squid-6.6-1.x86_64.rpm
    ```

   - Vérifiez que Squid fonctionne correctement après l'installation.
