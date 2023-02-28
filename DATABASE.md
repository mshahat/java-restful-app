# Configuring a database

## Local

### MySQL

Running a MySQL instance can easily be done by using `docker compose`:

```bash
$ docker compose up -d
```

> Note: If you start the application locally please be sure that `local` profile is active.


## Kubernetes 

> Integrating with Tanzu Application Platform (TAP)

### MySQL

Using the `config/workload.yaml` it is possible to build, test and deploy this application onto a
Kubernetes cluster that is provisioned with Tanzu Application Platform (https://tanzu.vmware.com/application-platform).

As with the local deployment a MySQL instance needs to be available at the cluster.
When using VMware Tanzu SQL with MySQL for Kubernetes (https://tanzu.vmware.com/sql and https://docs.vmware.com/en/VMware-Tanzu-SQL-with-MySQL-for-Kubernetes/index.html),
one could apply for an instance, and it will be automatically provisioned.

> Note: The Tanzu MySQL Operator must be installed in the cluster.

```bash
$ kubectl apply -n <your-services-namespace> -f config/service-operator/mysql.yaml
```

When the MySQL instance is created, resource binding needs to be configured in order that your workload can access
the MySQL instance, which maybe be in another namespace than your application workload.

1. If the `ClusterRole` and `ClusterInstanceClass` for service binding has not been defined, then as a service operator, you can run the following command.

   ```bash
   $ kubectl apply -f config/service-operator/mysql-cluster-role.yaml
   $ kubectl apply -f config/service-operator/mysql-cluster-instance-class.yaml
   ```

2. Configure the `ResourceClaimPolicy` to define which namespaces could have access to your MySQL instance namespaces.
   (optional: only needed when MySQL instance is in different namespace than workload namespace).

   Edit `consumingNamespaces` in `config/app-operator/mysql-resource-claim-policy.yaml` to contain your workload namespace and apply:
   
   ```bash
   $ kubectl apply -n <your-services-namespace> -f config/app-operator/mysql-resource-claim-policy.yaml
   ```

3. Create the `ResourceClaim` to be consumed by your workload that references your MySQL instance:
   > Note: change the `spec.ref.namespace` of `config/app-operator/mysql-resource-claim.yaml` to where the MySQL instance is deployed.
   
   ```bash
   $ kubectl apply -n <your-workload-namespace> -f config/app-operator/mysql-resource-claim.yaml
   ```

Now that the resource binding is configured the `Workload` can be applied.

