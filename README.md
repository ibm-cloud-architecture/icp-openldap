## OpenLDAP

Installs OpenLDAP and phpLDAPadmin with a small number of initial users for the purposes of demonstrating LDAP integration capabilities of ICP.


## Installation
To install the chart, you'll need the [helm cli](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0/app_center/create_helm_cli.html?view=kc) and the [IBM Cloud Private CLI](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0/manage_cluster/install_cli.html?view=kc)

1. Get the source code of the helm chart
   `git clone https://github.com/ibm-cloud-architecture/icp-openldap.git`
2. Package the helm chart using the helm cli
   `helm package icp-openldap`
3. If you have not, log in to your cluster from the IBM® Cloud Private CLI and log in to the Docker private image registry.
   `bx pr login -a https://<cluster_CA_domain>:8443 --skip-ssl-validation`
    Where cluster_CA_domain is the certificate authority (CA) domain. If you did not specify a CA domain, the default value is mycluster.icp. 
4. Install the Helm chart
   `bx pr load-helm-chart --archive <helm_chart_archive> [--clustername <cluster_CA_domain>]`
   Where helm_chart_archive is the name of your compressed Helm chart file and <cluster_CA_domain> is the certificate authority (CA) domain.
5. Update the package repository by using the IBM® Cloud Private cluster management console.
   1. From the IBM® Cloud Private management console, click ***Menu > Admin > Repositories***.
   2. Click Sync Repositories.
   3. Click ***Menu > Catalog*** The new Helm charts load into the Catalog, and you can install them into your cluster.

## Assets

Kubernetes Assets in this chart.

**OpenLDAP**
OpenLDAP

see details in [official site](http://www.openldap.org/)

default values below

```
OpenLdap:
  Image: "docker.io/osixia/openldap"
  ImagePullPolicy: "Always"
  Component: "openldap"

  Replicas: 1

  Cpu: "512m"
  Memory: "200Mi"

  Domain: "local.io"
  AdminPassword: "admin"
  Https: "false"
  SeedUsers: 
    usergroup: "icpusers"
    userlist: "user1,user2,user3,user4"
    initialPassword: "ChangeMe"
```

**phpLDAPadmin**
LDAP admin UI

see details in [official site](http://phpldapadmin.sourceforge.net/)

default values below
```
PhpLdapAdmin:
  Image: "docker.io/osixia/phpldapadmin"
  ImageTag: "0.7.0"
  ImagePullPolicy: "Always"
  Component: "phpadmin"

  Replicas: 1

  NodePort: 31080

  Cpu: "512m"
  Memory: "200Mi"
```

## Setup IBM Cloud Private LDAP integration

Detailed information about LDAP support in ICP avilable on the [IBM KnowledgeCenter](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0/user_management/configure_ldap.html)

After the chart is deployed, follow these steps to setup LDAP authentication
 
 1. From the helm release page take note of the OpenLDAP cluster ip and port for your deployment
 2. Navigate to ***ICP > Security > LDAP*** connection and insert the following details
    #### LDAP Connection
    - Name: `ldap`
    - Type: `Custom`
    - ID: `openLDAP`
    - Realm: `openLDAPRealm`
    - Server host: `<cluster-ip>`
    - Port: `389`
    
    #### LDAP authentication
    - Base DN: `dc=local,dc=io` (default value, adjust as needed)
    - Bind DN: `cn=admin,dc=local,dc=io` (default value, adjust as needed)
    - Admin Password: `admin` (default value, adjust as needed)
    
    #### LDAP Filters
    - Group filter: `(&(cn=%v)(objectclass=groupOfUniqueNames))`
    - User filter: `(&(uid=%v)(objectclass=person))`
    - Group ID map: `*:cn`
    - User ID map: `*:uid`
    - Group member ID map: `groupOfUniqueNames:uniquemember`

    Click ***Save***
    
 3. Import users by navigating to ***ICP > Security > Users > Import Group***
    - Common Name(CN): `cn=icpusers` (default value, adjust as needed)
    - Organizational Unit(OU): `ou=groups` 
    

## Credit

Inspired by work done by the [Samsung Cloud Native Computing Team](https://github.com/samsung-cnct) .
