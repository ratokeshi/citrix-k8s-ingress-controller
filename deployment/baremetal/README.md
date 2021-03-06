# Citrix Ingress Controller for kubernetes:
Citrix Ingress Controller is available in two flavours.
## Citrix Ingress Controller:
This runs as a POD that monitors the Kubernetes API server and configure NetScaler VPX and MPX. 

**YAML to be used:** ***citrix-k8s-ingress-controller.yaml***
## CPX with inbuilt Ingress Controller:
CPX with a builtin Citrix Ingress Controller agent that configures the CPX. CPX runs as pod and does N-S load balancing. 

**YAML to be used:** ***citrix-k8s-cpx-ingress.yaml***

## Install CPX with inbuilt Ingress Controller on Kubernetes:
   1. Apply the following command which pulls citrix cpx ingress controller and deploy it.
      ```
          kubectl apply -f  https://raw.githubusercontent.com/citrix/citrix-k8s-ingress-controller/master/deployment/baremetal/citrix-k8s-cpx-ingress.yml
      ```
   

## Install Citrix Ingress Controller on Kubernetes:
 1. Download the "citrix-k8s-ingress-controller.yaml" from the deployment Directory.
    ```
      wget  https://raw.githubusercontent.com/citrix/citrix-k8s-ingress-controller/master/deployment/baremetal/citrix-k8s-ingress-controller.yaml
    ```
                        
    This yaml has four section, in which first three is for cluster role creation and service account creation and the 
    next one is for citrix ingress controller pod creation. 
    * Cluster roles
    * Cluster role bindings
    * Service account
    * Citrix Ingress Controller pod creation
   
    First three are required for citrix ingress controller to monitor k8s events. No changes required.
    Next section defines environment variables required for Citrix Ingress Controller to configure the NetScaler.

 2. Update the following env variables, for Citrix Ingress Controller bringup.

    1. "Mandatory" Arguments:
       <details>
       <summary>NS_IP</summary>

         This is must for Citrix Ingress Controller to configure the NetScaler appliance. Provide,
         ```
            NSIP for standalone NetScaler  
            SNIP for HA (Management access has to be enabled) 
            CLIP for Cluster
         
         ```
       </details>
       <details>
       <summary>NS_USER and NS_PASSWORD</summary>

         This is for authenticating with NetScaler if it has non default username and password. We can directly pass username/password or use Kubernetes secrets.
         Please refer our [guide](https://github.com/citrix/citrix-k8s-ingress-controller/blob/master/docs/command-policy.md) for configuring a non default NetScaler username and password.
         
         Given Yaml uses k8s secrets. Following steps helps to create secrets to be used in yaml.

         Create secrets on Kubernetes for NS_USER and NS_PASSWORD
         Kubernetes secrets can be created by using 'kubectl create secret'.  

                 kubectl create secret  generic nslogin --from-literal=username='nsroot' --from-literal=password='nsroot'

         >**Note:** If you are using different secret name rather than nslogin, you have to update the "name" field in the yaml. 

       </details>
       <details>
       <summary>EULA</summary>

         This is end user license agreement which has to be YES for Citrix Ingress Controller to up and run.
                
       </details>
    2. "Optional" Arguments:

       <details>
       <summary>kubernetes_url</summary>

         This is an optional field for Citrix Ingress Controller to register for events. If user did not specify it explictly, citrix ingress controller use internal KubeAPIServer IP. 
   
       </details>
       <details>
       <summary>LOGLEVEL</summary>

         This is used for controlling the logs generated from Citrix Ingress Controller. Following options are available. By default log level is DEBUG. 
         * CRITICAL 
         * ERROR
         * WARNING
         * INFO
         * DEBUG
       </details>
       <details>

       <summary>NS_PROTOCOL and NS_PORT</summary>
                                
         These enviornment variables defines protocol and port used by Citrix Ingress Controller  to communicate with NetScaler.

         By default NS_PROTOCOL is https and NS_PORT is 443. Other option is to use HTTP and port 80. 
       </details>
       <details>
       <summary>Ingress Class</summary>

         [Ingress class](../../docs/configure/ingress-classes.md) is used when multiple Ingress Loadbalancers are used to load balance different ingress resources. 

         Citrix Ingress Controller will configure NetScaler only with the ingress classes listed under --ingress-classes

                     args:
                          - --ingress-classes
                                Citrix

         Ingress resources should have the same class mentioned:

                    annotations:
                          kubernetes.io/ingress.class: "Citrix"
       </details>
       <details>

       <summary>NS_VIP</summary>

       Citrix Ingress Controller will use the IP provided in this environment variable to configure a Vitual IP in the Tier-1 ADC which would recieve the application traffic from external world.

       This is useful in the case where all Ingress runs in the Virtual IP. This takes precedence over the [frontend-ip](../../docs/configure/annotations.md) annotation.

       **Usage:**
       
       ```
       - name: "NS_VIP"       
         value: "<Virtual IP address of Citrix ADC>"
       ```
       
       </details>
       <details>

       <summary>NS_APPS_NAME_PREFIX</summary>

       Citrix Ingress Controller will use the provided prefix to form the application entity name at Citrix ADC. 

       This is useful in the case, one Citrix ADC, load balancing applications from different cluster. Prefix allows to segregate the  kubernetes cluster configuration. Default value take as k8s_.
       **Usage:**
       
       ```
       - name: "NS_APPS_NAME_PREFIX"       
         value: "<Name of your choice>"
       ```
       </details>
       

3. Create using kubectl command. 

   Create Citrix Ingress Controller  on kubernetes by using 'kubectl create' command
        
           kubectl create -f citrix-k8s-ingress-controller.yaml

   This pulls the latest image and brings up the Citrix Ingress Controller.
                
   Official Citrix Ingress Controller docker images is <span style="color:red"> `quay.io/citrix/citrix-k8s-ingress-controller:1.1.1` </span>

4. #### Reachability to the Pod Network:
    For seamless functioning of services deployed in the Kubernetes cluster, it is essential that Ingress NetScaler device should be able to reach the underlying overlay network over which Pods are running. 
    `feature-node-watch` knob of Citrix Ingress Controller can be used for automatic route configuration on NetScaler towards the pod network. 
    Refer [Network Configuration](../../docs/network/staticrouting.md) for further details regarding the same. 
    By default, `feature-node-watch` is false. It needs to be explicitly set to true if auto route configuration is required.

