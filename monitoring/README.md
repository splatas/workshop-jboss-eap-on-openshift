# Workshop JBoss EAP on Openshift
## Monitoring

In order to show how monitoring works we developed an Application based on Quarkus and Micrometer.
We have 3 steps:
1. Installing Prometheus to collect information.

2. Installing Grafana to use dashboards

3. Adapt our application to generate metrics that will be shown on dashboards.


### Installing Prometheus to collect information.
1. Start the installation usiong Prometheus Operator...

     ![new-app-image](../images/new-app.png)

### Installing Grafana to use dashboards
1. Start the installation usiong Grafana Operator...

     ![new-app-image](../images/new-app.png)

2. Setting up Grafana.

     ![new-app-image](../images/new-app.png)

### Adapt our application

#### Prerrequisites
1. Maven:
    - You need Maven 3.6 installed on your machine. Go to the following link (https://maven.apache.org/download.cgi), download and install de version according to you OS.

2. Java 11:
    - You can follow this link (https://developers.redhat.com/openjdk-install) in order to install OpenJDK11.


3. If you want to start deveLoping Quarkus applications an easy way to do it is using JBANG.

    - Install:
    ```shell script
    curl -Ls https://sh.jbang.dev | bash -s - app install --fresh --force quarkus@quarkusio
    ```

    An error message is shown, so execute:

    ```shell script
    jbang trust add https://repo1.maven.org/maven2/io/quarkus/quarkus-cli/
    ```

    Something similar to this is shown: 
    ```shell script
    [jbang] Adding [https://repo1.maven.org/maven2/io/quarkus/quarkus-cli/] to /home/splatas/.jbang/trusted-sources.json
    ```


    Execute again the curl command and now you should see this: 
    ```shell script
    [jbang] Command installed: quarkus
    ```

    If you want to learn more about Quarkus, please visit its website: https://quarkus.io/ .

#### Process
1. Clone the repository:
    ```shell script
    git clone https://github.com/splatas/quarkus-metrics

    cd ./quarkus-metrics
    ```

2. Check files:
    - pom.xml:
        <dependency>
			<groupId>io.quarkus</groupId>
			<artifactId>quarkus-openshift</artifactId>
		</dependency>

		<dependency>
			<groupId>io.quarkus</groupId>
			<artifactId>quarkus-smallrye-openapi</artifactId>
		</dependency>
		
		<dependency>
		    <groupId>io.quarkus</groupId>
		    <artifactId>quarkus-micrometer-registry-prometheus</artifactId>
		</dependency>

    - Java class 'ExampleResource' (src/main/java/com/redhat/ExampleResource.java)


    - Some services: 'getSongs' and 'populateSongs'

3. Compile the application:

    ```shell script
    mvn quarkus:dev
    ```

     ![new-app-image](../images/new-app.png)

    This command compile, download dependencies and launch the application in DEV mode locally.

4. Deploy the application on Openshift cluster:

    - Login to cluster using credentials:
        ```shell script
        oc login --token=__TOKEN__ --server=https://api.cluster-58b1.dynamic.opentlc.com:6443
        ```    

    - Build, create objects and deploy the app on the cluster:
        ```shell script
        mvn clean package -Dquarkus.kubernetes-client.master-url=__CLUSTER-URL__ -Dquarkus.kubernetes-client.token=__TOKEN__


        mvn clean package -DskipTests -Dquarkus.kubernetes-client.master-url=https://api.cluster-58b1.dynamic.opentlc.com:6443 -Dquarkus.kubernetes-client.token=sha256~rsr0JvfMKJ4haHlilO52TSJ8JFEfNpR43MAv8DCaQJE -Dquarkus.kubernetes.deploy=true -Dquarkus.kubernetes-client.trust-certs=true -Dquarkus.openshift.route.expose=true 
        ```
         ![quarkus-build-deploy-on-ocp-image](../images/quarkus-build-deploy-on-ocp-image.png)

        ...

        ![quarkus-build-deploy-on-ocp-image-1](../images/quarkus-build-deploy-on-ocp-image-1.png)


    After that you should see an instance of the application running on your cluster:

    ![quarkus-app-deployed-on-ocp-image](../images/quarkus-app-deployed-on-ocp-image.png)


    - Working with the application:
        If you click on the exposed route you will see a page like this:

        Route: http://quarkus-metrics-eap-demo-sergio.apps.cluster-58b1.dynamic.opentlc.com/
        
        ![quarkus-app-main-page-image](../images/quarkus-app-main-page-image.png)
        

        
        You have an Endpoint to see the metrics of this app in raw format:
        http://quarkus-metrics-eap-demo-sergio.apps.cluster-58b1.dynamic.opentlc.com/q/metrics

        ![quarkus-app-metrics-image](../images/quarkus-app-metrics-image.png)

        
        Note that you can find several metrics related to resources used by the application, such as:
        - jvm_classes_loaded_classes
        - jvm_buffer_total_capacity_bytes
        - jvm_gc_memory_promoted_bytes_total
        - jvm_memory_max_bytes
        
        ... among others

    - Custom metrics:
        We added some specific annotations in the application code in order to generate custom metrics of our services.

        ![quarkus-app-getsongs-image](../images/quarkus-app-getsongs-image.png)

        ![quarkus-app-populatesongs-image](../images/quarkus-app-populatesongs-image.png)

        As you can see there are two annotations called:
            - @Timed
            - @Counted

        Just with those resources you can see metrics of those endpoints, generated easily. When you start to consume those endpoints, the metrics are available on [Metrics Endpoint](http://quarkus-metrics-eap-demo-sergio.apps.cluster-58b1.dynamic.opentlc.com/q/metrics)


        ![quarkus-app-getsongs-metrics-image.png](../images/quarkus-app-getsongs-metrics-image.png)

-
-
-


### FIN ------------------------
# code-with-quarkus Project

This project uses Quarkus, the Supersonic Subatomic Java Framework.

If you want to learn more about Quarkus, please visit its website: https://quarkus.io/ .

## Running the application in dev mode

You can run your application in dev mode that enables live coding using:
```shell script
./mvnw compile quarkus:dev
```

> **_NOTE:_**  Quarkus now ships with a Dev UI, which is available in dev mode only at http://localhost:8080/q/dev/.

## Packaging and running the application

The application can be packaged using:
```shell script
./mvnw package
```
It produces the `quarkus-run.jar` file in the `target/quarkus-app/` directory.
Be aware that it’s not an _über-jar_ as the dependencies are copied into the `target/quarkus-app/lib/` directory.

The application is now runnable using `java -jar target/quarkus-app/quarkus-run.jar`.

If you want to build an _über-jar_, execute the following command:
```shell script
./mvnw package -Dquarkus.package.type=uber-jar
```

The application, packaged as an _über-jar_, is now runnable using `java -jar target/*-runner.jar`.

## Creating a native executable

You can create a native executable using: 
```shell script
./mvnw package -Pnative
```

Or, if you don't have GraalVM installed, you can run the native executable build in a container using: 
```shell script
./mvnw package -Pnative -Dquarkus.native.container-build=true
```

You can then execute your native executable with: `./target/code-with-quarkus-1.0.0-SNAPSHOT-runner`

If you want to learn more about building native executables, please consult https://quarkus.io/guides/maven-tooling.

## Provided Code

### RESTEasy JAX-RS

Easily start your RESTful Web Services

[Related guide section...](https://quarkus.io/guides/getting-started#the-jax-rs-resources)
