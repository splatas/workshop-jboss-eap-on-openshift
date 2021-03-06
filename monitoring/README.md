# Workshop JBoss EAP on Openshift
## Monitoring

In order to show how monitoring works we developed an Application based on Quarkus and Micrometer.
We have 3 steps:
    
    - Install Prometheus to gather information.

    - Install Grafana to use dashboards

    - Configure the application to generate metrics that will be shown on dashboards.


### 1.Configure the application for metrics

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
    
    ```xml
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
    ```

    - Java class 'ExampleResource' (src/main/java/com/redhat/ExampleResource.java), services: 'getSongs' and 'populateSongs':

    ```java
        @Path("/q-metrics")
        public class ExampleResource {
	        ...
        
            @GET
            @Produces(MediaType.TEXT_PLAIN)
            @Path("/getSongs")
            @Timed("time_get_songs")
            @Counted
            public List<String> getSongs() {
                ...

            @GET
            @Produces(MediaType.TEXT_PLAIN)
            @Path("/populateSongs")
            @Timed("time_populate_songs")
            @Counted
            public List<String> populateSongs() {
                ...

    ```


3. Deploy the application locally:

    ```shell script
    mvn quarkus:dev
    ```

     ![quarkus-local-deploy](../images/quarkus-local-deploy.png)

    This command compile, download dependencies and launch the application in DEV mode locally.

4. Deploy the application on Openshift cluster:

    - Login to cluster using credentials:
        ```shell script
        oc login --token=__TOKEN__ --server=https://api.cluster-58b1.dynamic.opentlc.com:6443
        ```    

    - Build, create objects and deploy the app on the cluster:
        ```shell script
        mvn package -Dquarkus.kubernetes-client.master-url=__CLUSTER-URL__ -Dquarkus.kubernetes-client.token=__TOKEN__


        mvn package -DskipTests -Dquarkus.kubernetes-client.master-url=https://api.cluster-58b1.dynamic.opentlc.com:6443 -Dquarkus.kubernetes-client.token=sha256~rsr0JvfMKJ4haHlilO52TSJ8JFEfNpR43MAv8DCaQJE -Dquarkus.kubernetes.deploy=true -Dquarkus.kubernetes-client.trust-certs=true -Dquarkus.openshift.route.expose=true 
        ```
         ![quarkus-build-deploy-on-ocp-image](../images/quarkus-build-deploy-on-ocp-image.png)

        ...

        ![quarkus-build-deploy-on-ocp-image-1](../images/quarkus-build-deploy-on-ocp-image-1.png)


        After that you should see an instance of the application running on your cluster:

        ![quarkus-app-deployed-on-ocp-image](../images/quarkus-app-deployed-on-ocp-image.png)


    - Working with the application:
        
        If you click on the exposed route you will see a page like this:

        Route (example): http://quarkus-metrics-eap-demo-sergio.apps.cluster-58b1.dynamic.opentlc.com/
        
        ![quarkus-app-main-page-image](../images/quarkus-app-main-page-image.png)
        

        
        You have an Endpoint to see the metrics of this app in raw format:
        http://quarkus-metrics-eap-demo-sergio.apps.cluster-58b1.dynamic.opentlc.com/metrics

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

        Just with those resources you can see metrics of your endpoints, generated easily. When you start to consume those endpoints, the metrics are available on [Metrics Endpoint](http://quarkus-metrics-eap-demo-sergio.apps.cluster-58b1.dynamic.opentlc.com/q/metrics)


        ![quarkus-app-getsongs-metrics-image.png](../images/quarkus-app-getsongs-metrics-image.png)



    Go to Developer view / Monitoring / Metrics 

     ![prometheus-metrics-service-monitor.png](../images/prometheus-metrics-service-monitor.png)
   


### 2.Install Prometheus to gather information.

1. Start the installation using Prometheus Operator.

    Go to Admininistrator view / Operators / Operator Hub and select the Prometheus Operator.

    ![prometheus-operator](../images/prometheus-operator.png)

    Then install in your namespace:

    ![prometheus-install-form](../images/prometheus-install-form.png)

    ![prometheus-installing](../images/prometheus-installing.png)


2. Create a Prometheus instance and Service Monitor
    ```shell script
        oc create -f ./prometheus/prometheus-instance.yaml

        oc create -f ./prometheus/quarkus-metrics-service-monitor.yaml
    ```

    After this step you should see an instance of Prometheus running and a Service Monitor created.

    ![prometheus-instance](../images/prometheus-instance.png)

    ![prometheus-service-monitor](../images/prometheus-service-monitor.png)


    - Expose Prometheus service (just for testing):
        Prometheus instance not expose a route by default, so we can create one in order to access the Prometheus Graph page.

        ```shell script
            oc get svc | grep prometheus
        ```
        
        We need to expose the prometheus service create previously
        ```shell script
            oc expose svc $PROMETHEUS_SERVICE (shown with previous command)
        ```

        After that you should see a rout like this:
        
        http://prometheus-operated-eap-demo-sergio.apps.cluster-58b1.dynamic.opentlc.com/graph

        ![prometheus-page.png](../images/prometheus-page.png)
    
    - Verify the configuration:
        
        An easy way to validate the configuration is ok, click on 'Status / Targets' menu:

        ![prometheus-target-option.png](../images/prometheus-target-option.png)


        You should see a target like this:

        ![prometheus-targets.png](../images/prometheus-targets.png)


        ### WARNING: IF YOU DON'T SEE A TARGET DEFINED LIKE IN PREVIOUS IMAGE, YOU SHOULD VERIFY:
          - In Prometheus ServiceMonitor: 
            
            ####  SELECTOR / matchLables: (quarkus-metrics)
            ####  ENDPOINT / port: (http)

        ![prometheus-service-monitor-labels.png](../images/prometheus-service-monitor-labels.png)

        ### - Application Service LABEL:
            
        ![app-service-labels.png](../images/app-service-labels.png)


    - Metrics on Promehteus:

    Once you have configured the integration between the application and Prometheus, you should see the metrics on Promethues Dashboard:

    ![prometheus-dashboard.png](../images/prometheus-dashboard.png)


3. Enabling monitoring for user-defined projects (done):

     ```shell script
        oc create -f ./prometheus/cluster-monitoring-config.yaml

        oc create -f ./prometheus/user-workload-monitoring-config.yaml
     ```

    For more details, see this link: https://docs.openshift.com/container-platform/4.7/monitoring/enabling-monitoring-for-user-defined-projects.html



### 3.Install Grafana to use dashboards

1. Start the installation usiong Grafana Operator...

    Go to Admininistrator view / Operators / Operator Hub and select the Grafana Operator.

    ![grafana-operator](../images/grafana-operator.png)

    Then install in your namespace:

    ![grafana-install-form](../images/grafana-install-form.png)

    Use default values, except for 'Update Channel': select 'original'

    ![grafana-operator-form.png](../images/grafana-operator-form.png)


    ![grafana-installing](../images/grafana-installing.png)

2. Setting up Grafana.

    Once installed the Grafana Operator, we need to create an instance. So, go to Installed Operators / Grafana Operator / Create instance

    ![grafana-create-instance.png](../images/grafana-create-instance.png)


    Complete the field name and click create:

    ![grafana-create-instance-form.png](../images/grafana-create-instance-form.png)


    The instance is created with the following default credentials:
    
    ##User: root
    
    ##Password: secret

    Login on Grafana using the route, similar to this: https://grafana-route-eap-demo-sergio.apps.cluster-58b1.dynamic.opentlc.com/

    1. Create a Datasource:

        Click on left menu, 'Configuration / Data source':
    
        ![grafana-new-datasource.png](../images/grafana-new-datasource.png)
    
        Click on Add datasource and select type 'Prometheus':

        ![grafana-datasource-prometheus.png](../images/grafana-datasource-prometheus.png)


        Create the configuration with this parameters:
        - Name: a name to identify the datasource 
        - URL: http://PROMETHEUS_SERVICE_NAME:9090  (i.e.: http://prometheus-operated:9090)

        ... and click 'Save and Test'.

    ![grafana-new-datasource-config.png](../images/grafana-new-datasource-config.png)

    2. Import a Dashboard:

         Click on left menu, 'Configuration / + (Plus simbol) / Import': 14370

        ![grafana-import-dashboard.png](../images/grafana-import-dashboard.png)

        
        The id 14370 refers to a JVM Quarkus - Micrometer Metrics Dashboard. For more details please see the following link:
        https://grafana.com/grafana/dashboards/14370


        Click 'Upload' button and then select the datasource created previously 'Prometheus', and then 'Import':
        ![grafana-import-dashboard-14370.png](../images/grafana-import-dashboard-14370.png)

        
        Details:
        ![grafana-import-dashboard-14370-details.png](../images/grafana-import-dashboard-14370-details.png)
        




### FIN ------------------------
