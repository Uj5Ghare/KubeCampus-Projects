<div align="center"> <h1> Kasten K10 Restoring Demo </h1></div>
<br>

## Add the Kasten K10 helm repo:
```bash
helm repo add kasten https://charts.kasten.io/
```
<br>

## Install MySQL and Create a Demo Database
To experiment with backup and recovery of a cloud-native application, we will install MySQL and create a database in this step.

### Install MySQL
```bash
kubectl create namespace mysql
helm install mysql bitnami/mysql --namespace=mysql
watch -n 2 "kubectl -n mysql get pods"
```
<br>

### Create a Local Database
```sql
MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace mysql mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)
kubectl exec -it --namespace=mysql $(kubectl --namespace=mysql get pods -o jsonpath='{.items[0].metadata.name}') \
  -- env MYSQL_PWD=$MYSQL_ROOT_PASSWORD mysql -u root -e "CREATE DATABASE k10demo"
```
<br>

## Install K10 and Configure Storage

### Install Kasten K10

```bash
helm install k10 kasten/k10 --namespace=kasten-io --create-namespace
watch -n 2 "kubectl -n kasten-io get pods"
```

<br>

### Configure the Local Storage System
```bash
kubectl annotate volumesnapshotclass csi-hostpath-snapclass k10.kasten.io/is-snapshot-class=true
```

<br>

### Expose the K10 Dashboard
While not recommended for production environments, let’s set up access to the K10 dashboard by creating a NodePort.

```yaml
cat > k10-nodeport-svc.yaml << EOF
apiVersion: v1
kind: Service
metadata:
  name: gateway-nodeport
  namespace: kasten-io
spec:
  selector:
    service: gateway
  ports:
  - name: http
    port: 8000
    nodePort: 32000
  type: NodePort
EOF
```
```bash
kubectl apply -f k10-nodeport-svc.yaml
```

<br>

## Create a Policy to Backup MySQL
From the main K10 dashboard, click on the Policies card. There should be no policies visible at this point.

### Create Policy
- Click Create New Policy and Give the policy the name: mysql-backup
- Select Snapshot for the Action
- Select Hourly for the Action Frequency
- Leave the Snapshot Retention selection as-is
- Select By Name for Select Applications and then, from the dropdown, select mysql
- Leave all other settings as-is and select Create Policy

<br>

### Running the Policy
- The above policies will only run at the scheduled time (by default, at the top of the hour).
- To run the policies manually for the first time, click on Run Once on the Policies page.
- Confirm by clicking Run Policy and then go back to the main dashboard to view the job in action.
- Verify that the job has successfully completed.

<br>

## Simulate Accidental Data Loss and Recover the System
Now that we have a MySQL backup, let’s go simulate accidental data loss and then recover the system from that loss.
Verify that the database has been deleted by running:

```sql
MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace mysql mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)
kubectl exec -it --namespace=mysql $(kubectl --namespace=mysql get pods -o jsonpath='{.items[0].metadata.name}') -- env MYSQL_PWD=$MYSQL_ROOT_PASSWORD mysql -u root -e "SHOW DATABASES LIKE 'k10demo'"
```
<br>

## Recovering Data
- Go to the K10 dashboard
- click Applications
- select Restore on the MySQL card.
- Click on a recent restore point and then select the Exported restore point (this is stored in the object storage system instead of as a non-durable snapshot on the storage system).
-  In this case, we will select the default Application Name option to restore in place (Restore as “mysql”).
-  Leave all other selections as-is, click on Restore, and confirm the action.
- Run the following command in the terminal to view the restored database:

```bash
kubectl exec -it --namespace=mysql $(kubectl --namespace=mysql get pods -o jsonpath='{.items[0].metadata.name}') -- env MYSQL_PWD=$MYSQL_ROOT_PASSWORD mysql -u root -e "SHOW DATABASES LIKE 'k10demo'"
```

<br><br>
