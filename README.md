# Projet Kubernetes - Application de Vote

Ce projet déploie une application de vote sur un cluster Kubernetes. Il inclut des services, des déploiements, des configurations de stockage, des secrets, et un autoscaler pour assurer l'évolutivité de l'application.

## Structure du projet

Voici une description des fichiers YAML inclus dans ce projet et leur rôle :

1. serviceRedis.yaml

    Description : Définit un service Kubernetes pour exposer le déploiement Redis sur le cluster.
    Type de service : ClusterIP
    Port exposé : 6379

2. serviceVote.yaml

    Description : Définit un service Kubernetes pour l'application de vote, qui communique via le port 80.
    Type de service : ClusterIP
    Port exposé : 80

3. storageClass.yaml

    Description : Définit une classe de stockage pour provisionner dynamiquement des volumes persistants sur Azure.
    Provisioner : kubernetes.io/azure-disk
    Options supplémentaires :
        Expansion de volume activée.
        Type de stockage : Standard_LRS.

4. vote.yaml

    Description : Déploie l'application de vote en tant que déploiement Kubernetes.
    Détails :
        Réplique : 1
        Conteneur : celiaoued/vote-app:TAG (remplacez TAG par la version appropriée).
        Variables d'environnement :
            REDIS : Adresse du service Redis.
            STRESS_SECS : Intervalle de stress.
            REDIS_PWD : Mot de passe pour Redis (géré via un secret Kubernetes).

5. autoscaler.yaml

    Description : Implémente un autoscaler horizontal pour le déploiement de l'application de vote.
    Détails :
        Répliques minimum : 1
        Répliques maximum : 8
        Cible d'utilisation CPU : 70 %.

6. ingress.yaml

    Description : Configure un Ingress pour l'application de vote, en utilisant Azure Application Gateway.
    Détails :
        Point d'entrée : /
        Redirige vers le service votecluster via le port 80.

7. pvc.yaml

    Description : Définit une revendication de volume persistant (PVC) pour l'application Redis.
    Détails :
        Mode d'accès : ReadWriteOnce
        Taille demandée : 1Gi
        Classe de stockage : storageclass.

8. redis.yaml

    Description : Déploie un serveur Redis en tant que déploiement Kubernetes.
    Détails :
        Réplique : 1
        Conteneur : redis:latest
        Stockage persistant : Monté via un PVC.
        Sécurité : Mot de passe Redis injecté via un secret Kubernetes.

9. secretRedis.yaml

    Description : Définit un secret Kubernetes pour stocker le mot de passe Redis.
    Important : Ce fichier contient des informations sensibles. Ne le partagez pas publiquement.

## Prérequis

- Un cluster Kubernetes fonctionnel.
- Accès au cluster via kubectl.
- Configuration des permissions pour créer des déploiements, services, ingress, secrets, etc.
- Provisionnement Azure pour le stockage (si applicable).

## Instructions de déploiement

Déploiement des ressources : Appliquez chaque fichier YAML dans l'ordre suivant : <br/>

```code
kubectl apply -f storageClass.yaml <br/>
kubectl apply -f pvc.yaml <br/>
kubectl apply -f secretRedis.yaml <br/>
kubectl apply -f redis.yaml <br/>
kubectl apply -f serviceRedis.yaml <br/>
kubectl apply -f vote.yaml <br/>
kubectl apply -f serviceVote.yaml <br/>
kubectl apply -f autoscaler.yaml <br/>
kubectl apply -f ingress.yaml <br/>
```

Vérification des déploiements : Vérifiez que tous les pods, services et ingress sont en cours d'exécution :
```code
kubectl get all
```

Accès à l'application : Une fois le Ingress configuré, accédez à l'application via le point d'entrée configuré dans le fichier ingress.yaml.

## Notes importantes

Sécurité : Assurez-vous que le fichier secretRedis.yaml est correctement protégé et non partagé publiquement. <br/>
Personnalisation : Remplacez TAG dans vote.yaml par la version correcte de l'image Docker. <br/>
Stockage : Vérifiez que votre environnement Kubernetes prend en charge les disques Azure pour le provisionnement du stockage. <br/>
