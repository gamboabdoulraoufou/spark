

Cet article est écrit en _mars 2015._  
En ce moment la version actuelle de _Spark_ est **1.3.0**  

**_L'article couvre les points suivants:_**
- Installation des pré-requis
- Création et installation de spark
- Configuration basique de spark
- Deploiement de spark sur un cluster

**_Caractérisques:_**
- 2 VM sur Google Compute Engine
- OS: Ubuntu 12.04
- OpenJDK 1.6.0_27
- Scala 2.9.3
- Maven 3.0.4
- Python 2.7.3 
- Git 1.7.9.5 
  
  
### Installation des pré-requis
**Créer un utilusateur sparkmanager**
```sh
sudo adduser sparkmanager # mot de passe: spark
```

**Désactiver le mot de passe du compte utilisateur sparkmanager**
```sh
sudo sh -c "echo 'sparkmanager ALL=NOPASSWD: ALL' >> /etc/sudoers"
```

**Connecter en tant que sparkmanager**
```sh
su - sparkmanager # Password is spark
```

**Générer une clé ssh sur le noeud maitre**
```sh
ssh-keygen -t rsa -P ""
```

**Copier la clé publique sur les autres machines du cluster**  
Cela se fait par l'ajour de la clé dans les métadonnées
```sh
vim ~/.ssh/id_rsa.pub
```
**Install Oracle's JDK6**
```sh
# you may or may not want to remove open-jdk (not necessary):
sudo apt-get purge openjdk*

# to add PPA source repository to apt:
sudo add-apt-repository ppa:webupd8team/java

# to refresh the sources list:
sudo apt-get update

# to install JDK 6:
sudo apt-get install oracle-java6-installer
```

**Vérifier si java pointe JRE**
```sh
cd /etc/alternatives
ls -lat | grep java
```

... vous devez avoir quelque chose qui ressemble à ça

![java default](https://raw.github.com/mbonaci/mbo-spark/master/resources/java-default.PNG)


**Verifier `JAVA_HOME`**  
```sh
# Verifier où pointe java
echo $JAVA_HOME
# Si vous installer java pour la première fois
# JAVA_HOME ne retourne rien

# Configurer JAVA_HOME pour une nouvelle installation de java
echo "JAVA_HOME=/usr/lib/jvm/java-6-oracle/" >> /etc/environment # ~/.pam_environment
# Charger
source /etc/environment

# Configurer JAVA_HOME pour une ancienne installation
echo "JAVA_HOME=/usr/lib/jvm/java-6-oracle/" >> ~/.pam_environment

# Charger
source ~/.pam_environment

# Verifier à nouveau où pointe java
echo $JAVA_HOME
```

> `.pam_environment` est le nouveau `.bashrc`. [Pourquoi?](https://help.ubuntu.com/community/EnvironmentVariables#Session-wide_environment_variables)
  

**Installer Scala**
```sh
# Installation de Scala
sudo apt-get remove scala-library scala
wget http://www.scala-lang.org/files/archive/scala-2.11.4.deb
sudo dpkg -i scala-2.11.4.deb
sudo apt-get update
sudo apt-get install scala 
```

```sh
# Vérifier si l'installation est bien faite
scala -version

# Vous devez avoir:
Scala code runner version 2.11.4 -- Copyright 2002-2011, LAMP/EPFL
```

**Configurer `SCALA_HOME`**  

```sh
echo "SCALA_HOME=/usr/share/java" >> ~/.pam_environment

# Tout comme avec java on charge scala
source ~/.pam_environment

# On affiche le chemin de scala
echo $SCALA_HOME
```

**Installer Maven**

```sh
sudo apt-get install maven
```

**Installer GIT**

```sh
sudo apt-get install git
```

**Installer Spark**
```sh
# cloner le dossier
git clone git://github.com/apache/spark.git -b branch-1.3

# Ouvrir le dossier cloner
cd spark
```
**Construire spark**
```sh
# Configurer la memoire maven
export MAVEN_OPTS="-Xmx2g -XX:MaxPermSize=512M -XX:ReservedCodeCacheSize=512m"

# construire spark
build/mvn -Pyarn -Phadoop-2.4 -Dhadoop.version=2.4.0 -DskipTests clean package
```

**Lancer spark en mode standalone**
```sh
# lancer le noeud master du cluster standalone en localhost
./sbin/start-master.sh

# Lancer le workers
# /bin/spark-shell --master spark://IP:PORT
# Par exemple j'ai une machine dont le nom du hote est abd-spark-cluster
# je peux lancer cette machine sur mon cluster à l'aide de la commande suivante
/bin/spark-class org.apache.spark.deploy.worker.Worker spark://abd-spark-cluster:7077

# Connecter à http://IP:8080

# Lancer spark en mode interactif
./bin/spark-shell

# Quitter le mode interactif 
exit

# Arreter Spark
./sbin/stop-master.sh
```

Vous devez avoir quelque chose comme ça

![java default](https://raw.github.com/mbonaci/mbo-spark/master/resources/java-default.PNG)


**Ajouter 1 worker maitre et 4 workers slaves sur notre machine client (en loclahost)**
```sh
# Créer le fichier spark-env.sh
cp ./conf/spark-env.sh.template ./conf/spark-env.sh

# Ajouter le paramètre suivant dans le fichier spark-env.sh
echo "export SPARK_WORKER_INSTANCES=4" >> ./conf/spark-env.sh

# Lancer les workers slave
./sbin/start-slaves.sh
```

**Ajouter des workers au cluster standalone**
```sh
# Créer un ficher conf/slaves dans le worker maitre
nano conf/slaves
# Ajouter les hôtes des workers (1 hôte par ligne)
```

**Lancer les workers ajoutés précédemment**
```sh
# Lancer le worker master
./sbin/start-master.sh 

# Lancer les workers slave
./sbin/start-slaves.sh

# Lancer le tous les workers
./sbin/start-all.sh 
```

**Arreter les workers cluster standalone**
```sh
# Arreter le worker maitre
./sbin/stop-master.sh 

# Arreter les workers slave
./sbin/stop-slaves.sh 

# Arreter tous les workers
./sbin/stop-all.sh
```





