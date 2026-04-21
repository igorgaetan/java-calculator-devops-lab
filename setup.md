# 🎓 Guide de Formation DevOps
> **Java 21 • Maven / Gradle • SonarQube • Nexus**

---

## 🧭 Objectif du Lab
Mettre en place une chaîne CI/CD complète permettant de :
1. **Builder** une application Java.
2. **Analyser** la qualité du code avec SonarQube.
3. **Gérer** les artefacts (binaires) avec Sonatype Nexus.
4. **Comprendre** le cycle de vie des versions (Snapshot vs Release).

---

## 🖥️ PARTIE 1 — Environnement LOCAL (Poste Développeur)

### 1. Préparation du système
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git curl unzip -y
```

### 2. Installation de Java 21
```bash
sudo apt install openjdk-21-jdk -y
java -version
```

### 3. Outils de build
**Maven :**
```bash
sudo apt install maven -y
mvn -version
```

**Gradle 8.7 (Installation manuelle) :**
```bash
wget [https://services.gradle.org/distributions/gradle-8.7-bin.zip](https://services.gradle.org/distributions/gradle-8.7-bin.zip)
unzip gradle-8.7-bin.zip
sudo mv gradle-8.7 /opt/gradle

echo 'export PATH=$PATH:/opt/gradle/bin' >> ~/.bashrc
source ~/.bashrc
gradle -v
```

---

## ☁️ PARTIE 2 — Environnement SERVEUR (Docker)

### 1. Installation de Docker
```bash
sudo apt install docker.io -y
sudo usermod -aG docker $USER
# Note : Se déconnecter/reconnecter pour appliquer les groupes
```

### 2. Lancement de SonarQube
```bash
sudo docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  --restart always \
  -v sonarqube-data:/opt/sonarqube/data \
  sonarqube:lts-community
```
👉 Accès : `http://<SERVER_IP>:9000`

### 3. Lancement de Nexus
```bash
sudo docker run -d \
  --name nexus \
  -p 8081:8081 \
  --restart always \
  -v nexus-data:/nexus-data \
  sonatype/nexus3
```
👉 Accès : `http://<SERVER_IP>:8081`

**Récupérer le mot de passe admin initial :**
```bash
sudo docker exec nexus sh -c "cat /nexus-data/admin.password"
```

---

## 🚀 PARTIE 3 — Exécution du Projet

### 1. Cloner le projet
```bash
git clone [https://github.com/igorgaetan/java-calculator-devops-lab.git](https://github.com/igorgaetan/java-calculator-devops-lab.git)
cd java-calculator-devops-lab
```

### 2. Build & Qualité
**Analyse SonarQube (Maven) :**
```bash
mvn sonar:sonar \
  -Dsonar.login=admin \
  -Dsonar.password=VOTRE_PASS
```

---

## 📦 PARTIE 4 — Publication d'Artefacts (Nexus)

### 1. Configurer les accès (`~/.m2/settings.xml`)
Indispensable pour que Maven puisse s'authentifier sur votre Nexus privé :

```bash
cat << 'EOF' > ~/.m2/settings.xml
<settings>
  <servers>
    <server>
      <id>nexus-snapshots</id>
      <username>admin</username>
      <password>VOTRE_PASS_NEXUS</password>
    </server>
    <server>
      <id>nexus-releases</id>
      <username>admin</username>
      <password>VOTRE_PASS_NEXUS</password>
    </server>
  </servers>
</settings>
EOF
```

---

## 🧪 PARTIE 5 — TPE : Gestion des Versions (Crucial)

L'objectif de cette partie est de comprendre comment Nexus gère les différents états d'un logiciel.

### 🔹 Étape 1 — Le mode SNAPSHOT (Développement)
1. Dans le `pom.xml`, vérifiez que la version est `<version>1.0.0-SNAPSHOT</version>`.
2. Déployez : `mvn deploy`.
3. Modifiez le code et redéployez.
   * **Observation :** Nexus accepte les mises à jour. C'est une version de travail.

### 🔹 Étape 2 — Le mode RELEASE (Production)
1. Modifiez le `pom.xml` pour passer en version fixe : `<version>1.0.0</version>`.
2. Déployez : `mvn deploy`.
3. Tentez de redéployer la **même** version sans rien changer.
   * **Résultat :** Erreur `400 Bad Request`.
   * **Pourquoi ?** Une version Release est **immuable**. On ne modifie jamais un binaire déjà livré.

### 📊 Récapitulatif
| Type | Usage | Comportement Nexus |
| :--- | :--- | :--- |
| **SNAPSHOT** | Développement | Écrasable / Mutable |
| **RELEASE** | Livraison / Stable | Verrouillé / Immuable |

---

## 🧠 Points clés & Erreurs fréquentes
* **401 Unauthorized** : Vérifiez vos identifiants dans `settings.xml`.
* **400 Repository does not allow updating** : Tentative de modifier une RELEASE.
* **Ports** : Assurez-vous que les ports `9000` et `8081` sont ouverts sur votre serveur.

---

## 🎯 Conclusion
À la fin de ce TP, vous maîtrisez le flux complet :
**Code ➔ Build ➔ Test ➔ Analyse Qualité ➔ Stockage d'Artefact.**

---



## 🤖 Automatisation CI/CD (GitHub Actions)

Le projet inclut un pipeline complet défini dans `.github/workflows/main.yml`. À chaque **push** ou **pull request** sur la branche `main`, les étapes suivantes sont déclenchées :

1.  **Setup JDK 21** : Préparation de l'environnement avec la version Java requise.
2.  **Build & Tests** : Compilation du code et exécution des tests unitaires (`mvn clean verify`).
3.  **SonarQube Scan** : Analyse statique du code pour détecter les bugs et vulnérabilités.
4.  **Deploy to Nexus** : Si les étapes précédentes réussissent, l'artefact est publié sur le dépôt Nexus (Snapshot ou Release selon la version du `pom.xml`).

### 🔑 Configuration des Secrets
Pour faire fonctionner ce pipeline sur votre propre dépôt, vous devez configurer les secrets suivants dans GitHub :
* `SONAR_HOST_URL`
* `SONAR_TOKEN`
* `NEXUS_USERNAME`
* `NEXUS_PASSWORD`
```