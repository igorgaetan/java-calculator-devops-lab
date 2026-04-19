## 🎓 Guide de Formation : Setup Environnement DevOps (Java 21 / Sonar)

Ce guide détaille l'installation d'un environnement DevOps moderne pour Java 21 sur Ubuntu.

### 1. Préparation du système et dépendances
On installe les outils de base et on s'assure que le système est à jour.
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git curl unzip -y
```

### 2. Installation de Java 21 (JDK)
Nécessaire pour supporter les dernières fonctionnalités Java et les versions récentes de Gradle.
```bash
sudo apt install openjdk-21-jdk -y

# Vérification
java -version
```

### 3. Installation des outils de Build
#### A. Maven
La version fournie par `apt` est suffisante pour notre usage.
```bash
sudo apt install maven -y
```

#### B. Gradle 8.7 (Installation Manuelle)
**Important :** On évite `apt install gradle` car la version est trop ancienne pour Java 21.
```bash
wget https://services.gradle.org/distributions/gradle-8.7-bin.zip
unzip gradle-8.7-bin.zip
sudo mv gradle-8.7 /opt/gradle

# Ajout au PATH
echo 'export PATH=$PATH:/opt/gradle/bin' >> ~/.bashrc
source ~/.bashrc

# Vérification
gradle -v
```

### 4. Installation de Docker & SonarQube
Docker permet d'isoler le serveur SonarQube.
```bash
# Installation de Docker
sudo apt install docker.io -y
sudo usermod -aG docker $USER
# /!\ Déconnectez-vous et reconnectez-vous ici pour appliquer les droits

# Lancement de SonarQube
sudo docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  --restart always \
  -v sonarqube-data:/opt/sonarqube/data \
  -v sonarqube-extensions:/opt/sonarqube/extensions \
  -v sonarqube-logs:/opt/sonarqube/logs \
  sonarqube:lts-community

sudo docker run -d \
  --name nexus \
  -p 8081:8081 \
  --restart always \
  -v nexus-data:/nexus-data \
  sonatype/nexus3
```


sudo docker exec nexus \
  cat /nexus-data/admin.password

  

### 5. Exécution et Analyse du Projet
Clonez d'abord le projet :
```bash
git clone https://github.com/igorgaetan/java-calculator-devops-lab.git
cd java-calculator-devops-lab
```

#### Option Maven
```bash
# Test et couverture
mvn clean test
# Analyse (Utilisez votre Token généré sur l'interface Sonar)
mvn sonar:sonar -Dsonar.login=admin -Dsonar.password=VOTRE_PASS
```

#### Option Gradle
```bash
# Build et couverture
gradle clean build jacocoTestReport
# Analyse
gradle sonar -Dsonar.login=TON_TOKEN
```