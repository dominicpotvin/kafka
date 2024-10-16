# Vérifier les pods
kubectl get pods -n develop

# Vérifier les services
kubectl get services -n develop

# Voir les logs de Kafka
kubectl logs -f kafka-broker-0 -n develop

# Voir les logs de Zookeeper
kubectl logs -f zookeeper-0 -n develop