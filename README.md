

Cet article est écrit en _mars 2015._  
En ce moment la dernière version de _Spark_ etait **1.3.0**  

**_L'article couvre les points suivants:_**
- Installation des pré-requis
- Installation et construction de spark
- Lancement de spark en mode interactif (Scala et python)
- Deploiement de spark sur un cluster Standalone avec un ou plusieurs noeud
- Création et lancement d'une application Spark

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

### Installation et construction de Spark

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


### Lancement de spark en mode interactif (Scala et python)

**Spark en mode insteractif**
```sh
# Lancer spark en mode interactif Scala
./bin/spark-shell

# Quitter le mode interactif Scala
exit ()

# Lancer spark en mode interactif Python
./bin/pyspark-shell

# Quitter le mode interactif Python
exit ()
```

### Deploiement de Spark sur un cluster Standalone avec un ou plusieurs noeud

**Deployer un cluster spark en mode standalone sur un seul noeud (avec 1 ou plusieurs workers)**
```sh
# lancer le noeud master du cluster spark
./sbin/start-master.sh

# Aller à http://IP:8080

# Lancer des workers sur le cluster (tous les workers sont sur 1 seule machine)
# Créer un fichier spark-env.sh dans le dossier conf
cp ./conf/spark-env.sh.template ./conf/spark-env.sh

# Ajouter le paramètre suivant dans le fichier spark-env.sh
echo "export SPARK_WORKER_INSTANCES=4" >> ./conf/spark-env.sh

# Lancer les worker slaves du cluster
./sbin/start-slaves.sh

# Aller à http://IP:8080

# Arrêtr les workers slave
./sbin/stop-slaves.sh

# Arrêter le noeud maitre du cluster
./sbin/stop-master.sh
```

**Déployer manuellement un cluster spark en mode standalone sur un noeud (noeud maitre et worker)**
```sh
# lancer le noeud master
./sbin/start-master.sh

# Aller à http://IP:8080

# Ajouter manuellement un worker au cluster
# ./bin/spark-class org.apache.spark.deploy.worker.Worker spark://IP:PORT 
# Par exemple j'ai 1 machine dont le nom du hote sont: abd-spark-cluster
# je peux lancer cette machine sur le cluster à l'aide de la commande suivante
./bin/spark-class org.apache.spark.deploy.worker.Worker spark://abd-spark-cluster:7077

# Aller à http://IP:8080

# Arreter le cluster
./sbin/stop-master.sh
```

**Déployer automatiquement un cluster spark en mode standalone sur un ou plusieurs noeuds (workers)**
```sh
# Créer un ficher conf/slaves dans le worker maitre
nano conf/slaves
# Ajouter les hôtes des workers (1 hôte par ligne)

# Lancer le worker master
./sbin/start-master.sh 

# Aller à http://IP:8080

# Lancer les workers slave
./sbin/start-slaves.sh

# Aller à http://IP:8080

# Arreter le cluster
./sbin/stop-all.sh
```

### Création et lancement d'une application Spark

**Application spark**  
Créer un ficher SimpleApp.py et ajouter lui le code ci-dessous
```python
"""SimpleApp.py"""
from pyspark import SparkContext

# Initialiser Spark
appName = 'Simple App'
master = 'local' # Deploiement en local
# master = 'spark://abd-spark-cluster:7077' # Deploiement en mode cluster

conf = SparkConf().setAppName(appName).setMaster(master)
sc = SparkContext(conf=conf)

# Chargement des données
logFile = "/home/sparkmanager/spark/README.md"  
logData = sc.textFile(logFile).cache()

numAs = logData.filter(lambda s: 'a' in s).count()
numBs = logData.filter(lambda s: 'b' in s).count()

print "Lines with a: %i, lines with b: %i" % (numAs, numBs)
```

**Lancer l'application spark en mode local**
```sh
# Lancer spark
./sbin/start-master.sh 

# Exécuter le fichier SimpleApp.py
./bin/spark-submit --master local[2] SimpleApp.py

# Arrêter spark
./sbin/stop-master.sh 
```

**Lancer l'application spark en mode cluster**
```sh
# Lancer le cluster
./sbin/start-all.sh 

# Exécuter le fichier SimpleApp.py
./bin/spark-submit --master spark[5] SimpleApp.py

# Arrêter le cluster
./sbin/stop-all.sh 
```
