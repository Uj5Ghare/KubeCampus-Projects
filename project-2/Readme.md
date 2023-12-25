# Run a Stateful Application Using MySQL

This guide will walk you through the process of running a stateful application using MySQL on Kubernetes.

## Prerequisites

- Git
- Kubernetes
- Helm

## Steps

1. Clone the repository and navigate to the project directory:
```bash
git clone https://github.com/Uj5Ghare/KubeCampus-Projects.git
cd KubeCampus-Projects/Project-2/
export REPOSITORY_PREFIX=ahmedgabercod
```

2. Apply the initial namespace and services:
```bash
kubectl apply -f k8s/init-namespace/
kubectl apply -f k8s/init-services
```

3. We will use Helm to setup the MySQL backend for each service. Add the bitnami repository and update Helm:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

4. Install the database instance for the vets service:
```bash
helm install vets-db-mysql bitnami/mysql --namespace spring-petclinic --version 9.1.4 --set auth.database=service_instance_db
```

5. Install the database instance for the visits service:
```bash
helm install visits-db-mysql bitnami/mysql --namespace spring-petclinic  --version 9.1.4 --set auth.database=service_instance_db
```

6. Install the database instance for the customers service:
```bash
helm install customers-db-mysql bitnami/mysql --namespace spring-petclinic  --version 9.1.4 --set auth.database=service_instance_db
```

7. Deploy to Kubernetes:
```bash
./scripts/deployToKubernetes.sh
```

8. Verify the pods are deployed:
```bash
watch kubectl get pods -n spring-petclinic
```
9. Export the MySQL password:
```bash
export MYSQL_PASSWORD=$(kubectl get secret customers-db-mysql -n spring-petclinic -o jsonpath='{.data.mysql-root-password}' | base64 -d)
```

10. Run the MySQL client:
```bash 
kubectl run mysql-client --image=mysql:8.0 --namespace spring-petclinic -i --rm --restart=Never --\
  mysql -h customers-db-mysql -uroot -p$MYSQL_PASSWORD<<EOF
USE service_instance_db;
SELECT * FROM owners;
EOF
```

11. Scale the deployments:
```bash 
kubectl scale deployments -n spring-petclinic {api-gateway,customers-service,vets-service,visits-service} --replicas=3
```

12. Verify the deployments are scaled:
```bash
kubectl get pods -n spring-petclinic
```