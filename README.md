# Implement Aqua on ARO (Azure RedHat OpenShift)

## Introduction 
Follow the procedure on this page to perform a standard deployment of Aqua CSP in an OpenShift cluster. The Aqua Server components are deployed as Pods and Services, while the Aqua Enforcer is deployed as a DaemonSet.

This procedure describes how to deploy Aqua components on OpenShift through the OpenShift command line (oc utility). It assumes that the Aqua images will be used directly from the private Aqua Security registry on Docker Hub.

## Step 1: Prepare prerequisites
Perform the following prerequisite steps before you deploy Aqua components.

1. Login in to the cluster as a user with *ARO Customer Admin* privileges

```
oc login -u <user>
```

2. Create a new project and account for the Aqua Server components: aqua-web, aqua-gateway, and aqua-db. 

```
oc new-project aqua-security
oc create serviceaccount aqua-account -n aqua-security
```

3. Annotate the SCCs you're editing. This will prevent the Sync Pod from reverting your changes.
```
oc annotate scc hostaccess openshift.io/reconcile-protect=true
oc annotate scc privileged openshift.io/reconcile-protect=true
```

4. Set the Aqua'a enforcer priviliges 
```
oc adm policy add-cluster-role-to-user customer-admin-cluster system:serviceaccount:aqua-security:aqua-account
oc adm policy add-scc-to-user privileged system:serviceaccount:aqua-security:aqua-account
oc adm policy add-scc-to-user hostaccess system:serviceaccount:aqua-security:aqua-account
```

5. Create a secret to store the credential to pull Aqua's images from the registry. Replace the key holders below with the credential you recived from Aqua Security
```
oc create secret docker-registry aqua-registry --docker-server=registry.aquasec.com --docker-username=<AQUA_USERNAME> --docker-password=<AQUA_PASSWORD> --docker-email=no@email.com -n aqua-security
```

## Step 2: Deploy the Aqua Server, Database, and Gateway

1. Download the aqua-console.yaml file and make the following changes -
2. Replace <PUBLIC_IP> with the DNS name or IP address of your OpenShift master node. This address will be used to access the Aqua Server after deployment has been completed: http://<PUBLIC_IP>:30080. 
3. Replace all occurrences of <DB_PASSWORD> with a password of your choice
4. In case there are DNS resolution issues, you might need to replace all instances of aqua-db with the IP address of the Aqua Gateway service.




