## Simulation Gazebo

La simulation utilisée dans ce projet est basée sur le travail suivant :

👉 https://github.com/HKPolyU-UAV/E2ES/tree/master

Le framework **E2ES (End-to-End UAV Simulation Platform)** est une plateforme complète de simulation de drones développée sous Gazebo et ROS. Elle a été conçue pour la recherche en **SLAM visuel, navigation autonome et planification de trajectoire**.

### 📌 Description du framework utilisé

Le dépôt fournit un environnement de simulation UAV complet comprenant :

- Un monde 3D réaliste sous Gazebo pour les tests de navigation  
- Un modèle de drone de type MAV avec dynamique réaliste  
- Une caméra embarquée de type **RGB-D (RealSense)** permettant la perception de profondeur et la reconstruction de l’environnement  
- Des outils intégrés pour :
  - localisation (visual odometry / SLAM)
  - cartographie (mapping 3D)
  - planification de trajectoire autonome  

Ce type de configuration permet de tester des algorithmes de navigation complète **de bout en bout (end-to-end)** dans un environnement simulé réaliste.

### 📷 Caméra RGB-D et perception

Le système utilise une caméra RGB-D embarquée permettant de récupérer à la fois :
- une image RGB (vision classique)
- une carte de profondeur (depth)

Ces données sont utilisées pour la reconstruction 3D de l’environnement et les algorithmes de SLAM visuel.

### 🔧 Intégration dans notre projet

Dans notre projet, nous avons :

- Importé le **monde 3D (Gazebo world)** fourni dans E2ES pour nos propres tests de navigation  
- Réutilisé la structure de simulation UAV pour nos essais de contrôle et de trajectoire  
- Implémenté et adapté les scripts de simulation du drone afin de les intégrer à notre propre architecture de contrôle  
- Exploité la caméra RGB-D pour les tests de perception et de navigation autonome  
