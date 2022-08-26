# vac-seen-event-store
Creates a PostgreSQL database which will be used by Marten ([https://martendb.io/](https://martendb.io/)).

This is Part Two (of four) of the C#, Kafka and OpenShift activity.

## Need help?
If you need help or get stuck, email devsandbox@redhat.com.  
If you find a defect, create an Issue in this repository.

## Work environment  
This activity needs the following:
* Access to a command line
* You must be signed into your OpenShift sandbox

## Prerequisites and Requirements  
* `oc` command line tool must be installed  
* Access to OpenShift sandbox

## Create PostgreSQL instance

The goal is to create an instance of a PostgreSQL database in your Red Hat OpenShift cluster. This database is required by the Marten event store that will be used in the C# application.  

### Step 1: Find the needed template
Get a list the templates to find the PostgreSQL template that is needed. It turns out to be "postgresql-persistent". Use the following command to do this:

Bash:  
`oc get templates --namespace=openshift | grep postgresql`  

PowerShell:  
`oc get templates --namespace=openshift | Select-String postgresql`  

The results of the `oc get templates` command will be similar to this:  
```console
eap72-postgresql-persistent-s2i                     An example Red Hat JBoss EAP 7 application with a persistent PostgreSQL datab...   37 (16 blank)     11
eap72-postgresql-s2i                                An example Red Hat JBoss EAP 7 application with an PostgreSQL database config...   36 (16 blank)     10
jws31-tomcat7-postgresql-persistent-s2i             An example JBoss Web Server application with a PostgreSQL database. For more...    28 (10 blank)     10
jws31-tomcat7-postgresql-s2i                        Application template for JWS PostgreSQL applications built using S2I.              27 (10 blank)     9
jws31-tomcat8-postgresql-persistent-s2i             Application template for JWS PostgreSQL applications with persistent storage...    28 (10 blank)     10
jws50-tomcat9-postgresql-persistent-s2i             Application template for JWS PostgreSQL applications with persistent storage...    28 (10 blank)     10
jws53-openjdk11-tomcat9-postgresql-persistent-s2i   Application template for JWS PostgreSQL applications with persistent storage...    28 (10 blank)     10
jws53-openjdk8-tomcat9-postgresql-persistent-s2i    Application template for JWS PostgreSQL applications with persistent storage...    28 (10 blank)     10
nodejs-postgresql-example                           An example Node.js application with a PostgreSQL database. For more informati...   18 (4 blank)      8
nodejs-postgresql-persistent                        An example Node.js application with a PostgreSQL database. For more informati...   19 (4 blank)      9
postgresql-ephemeral                                PostgreSQL database service, without persistent storage. For more information...   7 (2 generated)   3
postgresql-persistent                               PostgreSQL database service, with persistent storage. For more information ab...   8 (2 generated)   4
processserver64-amq-postgresql-persistent-s2i       An example BPM Suite application with A-MQ and a PostgreSQL database. For mor...   46 (10 blank)     14
processserver64-amq-postgresql-s2i                  An example BPM Suite application with A-MQ and a PostgreSQL database. For mor...   44 (10 blank)     12
processserver64-postgresql-persistent-s2i           An example BPM Suite application with a PostgreSQL database. For more informa...   37 (11 blank)     10
rails-postgresql-example                            An example Rails application with a PostgreSQL database. For more information...   20 (4 blank)      8
rhpam710-kieserver-postgresql                       Application template for a managed KIE Server with a PostgreSQL database, for...   71 (42 blank)     10
rhpam711-kieserver-postgresql                       Application template for a managed KIE Server with a PostgreSQL database, for...   71 (42 blank)     9
rhpam77-kieserver-postgresql                        Application template for a managed KIE Server with a PostgreSQL database, for...   67 (41 blank)     10
rhpam78-kieserver-postgresql                        Application template for a managed KIE Server with a PostgreSQL database, for...   67 (41 blank)     10
rhpam79-kieserver-postgresql                        Application template for a managed KIE Server with a PostgreSQL database, for...   67 (41 blank)     10
sso72-postgresql                                    An example RH-SSO 7 application with a PostgreSQL database. For more informat...   33 (17 blank)     8
sso72-postgresql-persistent                         An example RH-SSO 7 application with a PostgreSQL database. For more informat...   34 (17 blank)     9
sso73-ocp4-x509-postgresql-persistent               An example application based on RH-SSO 7.3 image. For more information about...    21 (9 blank)      8
sso73-postgresql                                    An example application based on RH-SSO 7.3 image. For more information about...    34 (18 blank)     8
sso73-postgresql-persistent                         An example application based on RH-SSO 7.3 image. For more information about...    35 (18 blank)     9
sso74-ocp4-x509-postgresql-persistent               An example application based on RH-SSO 7.4 on OpenJDK image. For more informa...   21 (9 blank)      8
sso74-postgresql                                    An example application based on RH-SSO 7.4 on OpenJDK image. For more informa...   34 (18 blank)     8
sso74-postgresql-persistent                         An example application based on RH-SSO 7.4 on OpenJDK image. For more informa...   35 (18 blank)     9
```

### Step 2: Find the necessary parameters
Before the PostgreSQL instance can be created, the necessary parameters must be discovered. Use the following command to get the list of parameters need:  

`oc describe templates postgresql-persistent --namespace=openshift`  

The results will be similar to the following. Notice the list of parameters for the template and note that, in this case, we want to specify the user name, the user password, and the name of the database:  

```console
Name:           postgresql-persistent
Namespace:      openshift
Created:        16 months ago
Labels:         samples.operator.openshift.io/managed=true
Description:    PostgreSQL database service, with persistent storage. For more information about using this template, including OpenShift considerations, see https://github.com/sclorg/postgresql-container/.

                NOTE: Scaling to more than one replica is not supported. You must have persistent volumes available in your cluster to use this template.
Annotations:    iconClass=icon-postgresql
                openshift.io/display-name=PostgreSQL
                openshift.io/documentation-url=https://docs.okd.io/latest/using_images/db_images/postgresql.html
                openshift.io/long-description=This template provides a standalone PostgreSQL server with a database created.  The database is stored on persistent storage.  The database name, username, and password are chosen via parameters when provisioning this service.
                openshift.io/provider-display-name=Red Hat, Inc.
                openshift.io/support-url=https://access.redhat.com
                samples.operator.openshift.io/version=4.9.15
                tags=database,postgresql

Parameters:
    Name:               MEMORY_LIMIT
    Display Name:       Memory Limit
    Description:        Maximum amount of memory the container can use.
    Required:           true
    Value:              512Mi

    Name:               NAMESPACE
    Display Name:       Namespace
    Description:        The OpenShift Namespace where the ImageStream resides.
    Required:           false
    Value:              openshift

    Name:               DATABASE_SERVICE_NAME
    Display Name:       Database Service Name
    Description:        The name of the OpenShift Service exposed for the database.
    Required:           true
    Value:              postgresql

    Name:               POSTGRESQL_USER
    Display Name:       PostgreSQL Connection Username
    Description:        Username for PostgreSQL user that will be used for accessing the database.
    Required:           true
    Generated:          expression
    From:               user[A-Z0-9]{3}

    Name:               POSTGRESQL_PASSWORD
    Display Name:       PostgreSQL Connection Password
    Description:        Password for the PostgreSQL connection user.
    Required:           true
    Generated:          expression
    From:               [a-zA-Z0-9]{16}

    Name:               POSTGRESQL_DATABASE
    Display Name:       PostgreSQL Database Name
    Description:        Name of the PostgreSQL database accessed.
    Required:           true
    Value:              sampledb

    Name:               VOLUME_CAPACITY
    Display Name:       Volume Capacity
    Description:        Volume space available for data, e.g. 512Mi, 2Gi.
    Required:           true
    Value:              1Gi

    Name:               POSTGRESQL_VERSION
    Display Name:       Version of PostgreSQL Image
    Description:        Version of PostgreSQL image to be used (10-el7, 10-el8, or latest).
    Required:           true
    Value:              10-el8


Object Labels:  template=postgresql-persistent-template

Message:        The following service(s) have been created in your project: ${DATABASE_SERVICE_NAME}.

                       Username: ${POSTGRESQL_USER}
                       Password: ${POSTGRESQL_PASSWORD}
                  Database Name: ${POSTGRESQL_DATABASE}
                 Connection URL: postgresql://${DATABASE_SERVICE_NAME}:5432/

                For more information about using this template, including OpenShift considerations, see https://github.com/sclorg/postgresql-container/.

Objects:
    Secret                      ${DATABASE_SERVICE_NAME}
    Service                     ${DATABASE_SERVICE_NAME}
    PersistentVolumeClaim       ${DATABASE_SERVICE_NAME}
    DeploymentConfig            ${DATABASE_SERVICE_NAME}
```  

### Step 3: Create the PostgreSQL database instance
With the parameter information in hand, the following command will create exactly what is wanted — a persistent PostgreSQL instance in the project in the cluster. Again, notice that we are specifying the user name (POSTGRESQL_USER), the user password (POSTGRESQL_PASSWORD), and the database name (POSTGRESQL_DATABASE) parameters.

The user name, password, and database name will later be used in an OpenShift Secret object to enable a microservice to connect to this database.

Run the following command to create the PostgreSQL instance:

`oc new-app openshift/postgresql-persistent -e POSTGRESQL_USER=postgres -e POSTGRESQL_DATABASE=postgres -e POSTGRESQL_PASSWORD=7f986df431344327b52471df0142e520`  

### Next step(s)
The database has been created. The next step will be to create a program that uses this PostgreSQL instance for a Marten event store.