### 🚀 Résumé de l'exercice : Chaîne d'intégration locale (Java/Sonar)

**Objectif :** Mise en place d'une chaîne d'intégration continue pour une application Java, avec support bilingue **Maven** et **Gradle**.

**Points clés validés :**
* **Gestion des environnements :** J'ai configuré le projet pour fonctionner sous **Java 21**. Une attention particulière a été portée à l'installation de **Gradle 8.7**, car les versions par défaut fournies par le gestionnaire de paquets `apt` sur Ubuntu sont trop anciennes pour supporter les dernières versions du JDK.
* **Analyse de Qualité :** Intégration de **SonarQube** via Docker pour l'analyse statique du code (détection de la dette technique, bugs et vulnérabilités).
* **Sécurité & Automatisation :** Utilisation de **Tokens d'authentification** pour les scanners Maven/Gradle, évitant ainsi l'usage de mots de passe en clair.
* **Couverture de code :** Mise en œuvre de **JaCoCo** pour mesurer l'efficacité des tests unitaires JUnit 5.

**Difficulté rencontrée :** La principale contrainte a été l'obsolescence des paquets système (`apt install gradle`), résolue par une installation manuelle de la version 8.7 pour garantir la compatibilité avec le moteur Java actuel.