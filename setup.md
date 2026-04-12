
---

# 🎓 Guide de Formation : Setup Environnement DevOps (Java/Sonar)

Ce guide permet de configurer un environnement complet pour builder, tester et analyser la qualité d'une application Java.

## 📋 Prérequis (Instance EC2)
* **OS :** Ubuntu 22.04 LTS ou 24.04 LTS.
* **Type d'instance :** `t3.medium` recommandé (SonarQube nécessite au moins 4Go de RAM).
* **Sécurité :** Ouvrir le port **9000** (SonarQube) et **80** (optionnel) dans le Security Group.

---

## 1. Installation de Docker
Indispensable pour faire tourner SonarQube sans polluer le système hôte.

```bash
# Mise à jour des dépôts
sudo apt update && sudo apt upgrade -y

# Installation des dépendances
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y

# Ajout de la clé GPG et du dépôt officiel Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Installation de Docker
sudo apt update && sudo apt install docker-ce docker-ce-cli containerd.io -y

# Configuration post-installation (pour éviter 'sudo')
sudo usermod -aG docker $USER
newgrp docker
```

---

## 2. Installation de Java (JDK 17)
On utilise la version 17 LTS, standard actuel dans l'industrie.

```bash
sudo apt install openjdk-17-jdk -y

# Configuration de la variable d'environnement JAVA_HOME
echo 'export JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"' >> ~/.bashrc
source ~/.bashrc
```

---

## 3. Installation des outils de Build (Maven & Gradle)

### A. Maven
```bash
sudo apt install maven -y
```

### B. Gradle (Version 8.7)
```bash
wget https://services.gradle.org/distributions/gradle-8.7-bin.zip
sudo apt install unzip -y
unzip gradle-8.7-bin.zip
sudo mv gradle-8.7 /opt/gradle
echo 'export PATH=$PATH:/opt/gradle/bin' >> ~/.bashrc
source ~/.bashrc
```

---

## 4. Lancement de SonarQube
On utilise Docker pour déployer le serveur d'analyse en une seule ligne.

```bash
# Lancement du conteneur
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community
```
> **Note :** Attendez environ 60 secondes. Accédez ensuite à `http://votre-ip-ec2:9000`. 
> *(Login par défaut : admin / admin(or password). Vous devrez changer le mot de passe à la première connexion).*

---

## 5. Exécution du Projet

### Option 1 : Avec Maven
```bash
# Compiler et lancer les tests unitaires + couverture JaCoCo
mvn clean test

# Envoyer l'analyse vers SonarQube
# Remplacez par votre token généré dans l'interface Sonar
mvn sonar:sonar -Dsonar.login=admin -Dsonar.password=VOTRE_PASS
```

### Option 2 : Avec Gradle
```bash
# Builder et générer le rapport de couverture
gradle clean build jacocoTestReport

# Envoyer l'analyse vers SonarQube
gradle sonar -Dsonar.login=TON_TOKEN
```

---

## 📊 Interprétation des Résultats

Une fois les commandes terminées, retournez sur l'interface SonarQube :
1. **Quality Gate :** Si c'est au "Vert", le code respecte les standards.
2. **Coverage :** Indique le % de code testé par `CalculatorTest.java`.
3. **Bugs & Vulnerabilities :** Liste les failles de sécurité potentielles.
4. **Technical Debt :** Temps estimé pour corriger les "Code Smells".