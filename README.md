# Déploiement Kafka sur Kubernetes

Ce guide détaille les étapes pour déployer Kafka sur un cluster Kubernetes, effectuer des tests de base, et nettoyer l'environnement.

## Prérequis

- Kubernetes cluster opérationnel
- Helm 3 installé
- kubectl configuré pour votre cluster

## 1. Suppression des déploiements existants

Si vous avez des déploiements Kafka ou Zookeeper existants, supprimez-les d'abord :

```powershell
# Supprimer Kafka
helm uninstall kafka -n develop

# Supprimer Zookeeper
helm uninstall zookeeper -n develop

# Supprimer les PVC restants (ajustez les noms si nécessaire)
kubectl delete pvc -l app.kubernetes.io/instance=kafka -n develop
kubectl delete pvc -l app.kubernetes.io/instance=zookeeper -n develop

# Vérifier qu'il ne reste plus de ressources
kubectl get all -n develop
```

## 2. Installation et mise en place

### 2.1 Créer le namespace

```powershell
kubectl create namespace develop
```

### 2.2 Installer Zookeeper

```powershell
helm install zookeeper bitnami/zookeeper `
  --namespace develop `
  --values .\helm\zookeeper\values.yaml
```

### 2.3 Installer Kafka

```powershell
helm install kafka bitnami/kafka `
  --namespace develop `
  --values .\helm\kafka\values.yaml
```

### 2.4 Vérifier l'installation

```powershell
kubectl get pods -n develop
```

## 3. Tests

### 3.1 Créer des pods clients

```powershell
# Pod pour le producteur
kubectl run kafka-producer --restart='Never' --image docker.io/bitnami/kafka:3.8.0-debian-12-r5 --namespace develop --command -- sleep infinity

# Pod pour le consommateur
kubectl run kafka-consumer --restart='Never' --image docker.io/bitnami/kafka:3.8.0-debian-12-r5 --namespace develop --command -- sleep infinity
```

### 3.2 Test du producteur

Dans une nouvelle fenêtre PowerShell :

```powershell
kubectl exec --tty -i kafka-producer --namespace develop -- bash

# Dans le pod, lancez :
kafka-console-producer.sh --broker-list kafka-broker-0.kafka-broker-headless.develop.svc.cluster.local:9092 --topic test
# Tapez des messages et appuyez sur Entrée
```

### 3.3 Test du consommateur

Dans une autre fenêtre PowerShell :

```powershell
kubectl exec --tty -i kafka-consumer --namespace develop -- bash

# Dans le pod, lancez :
kafka-console-consumer.sh --bootstrap-server kafka.develop.svc.cluster.local:9092 --topic test --from-beginning
```

## 4. Nettoyage

Après les tests, nettoyez votre environnement :

```powershell
# Supprimer les pods de test
kubectl delete pod kafka-producer kafka-consumer -n develop

# Si vous voulez tout supprimer :
helm uninstall kafka zookeeper -n develop
kubectl delete pvc -l app.kubernetes.io/instance=kafka -n develop
kubectl delete pvc -l app.kubernetes.io/instance=zookeeper -n develop
kubectl delete namespace develop
```

## Notes

- Assurez-vous d'être dans le bon répertoire (E:\Projets\KafkaConnect_IOT) avant d'exécuter les commandes Helm.
- Les fichiers de configuration (`values.yaml`) pour Kafka et Zookeeper doivent être présents dans les dossiers respectifs sous `.\helm\`.
- Ajustez les versions des images et les configurations selon vos besoins spécifiques.
