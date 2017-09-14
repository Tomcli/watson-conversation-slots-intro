# watson-conversation-slots-intro-kube

This directory allows you to deploy the watson-conversation-slots-intro application into a container running on Bluemix. 

# Steps

## Create the Kubernetes cluster

* Follow the instructions to [Create a Kubernetes Cluster](https://github.com/IBM/container-journey-template). When you get to step 2 name your cluster Watson. 
```
$ bx cs cluster-create --name Watson
```
* If you choose a different name or already have a cluster, please substitute that name for 'Watson' in the instructions that follow.

## Create the Watson Conversation Service and Bind to your Cluster

Either follow the instructions to [Create a Conversation Service](https://console.ng.bluemix.net/catalog/services/conversation) or Perform the following from the Bluemix CLI:

* Create the Watson Conversation service.

```
bx service create conversation free 
	conversation-service-watson-pizzeria
```


* Verify that the service instance is created.

```
bx service list
```

* Obtain the ID of your cluster.

```
bx cs clusters
```

* Bind the service instance to your cluster.

```
bx cs cluster-service-bind <cluster-ID> default conversation-service-watson-pizzeria
```

## Load the Watson Conversation

* Follow the instructions to [Load the Watson Conversation](https://github.com/IBM/watson-conversation-slots-intro#4-configure-watson-conversation)

## Create a Kubernetes Configuration Map with the Workspace ID

* Create a Kubernetes Configuration Map

```kubectl create configmap watson-pizzeria-config --from-literal=workspace_id=<WorkSpace_ID>```

* Verify that the configuration is set

```kubectl get configmaps watson-pizzeria-config -o yaml```

## Deploy the Pod

* Deploy

```kubectl create -f watson-pizzeria-pod.yml```

* Identify the **Public IP** address of your worker. Here I've used the cluster name "Watson"

```bx cs workers Watson``` 

* Identify the external port your pod is listening on. Note: The pizza-bot container listens on port 3000. Kubernetes maps this a publicly addressable port.

```kubectl get services pizza-bot```

* Access the Watson Conversation Slots using `http://<IP Address>:<Port>`

## Look Under the Hood

Now that you have created and bound a service instance to your cluster, let's take a deeper look at what is happening behind the scenes.


* Binding the conversation-service-watson-pizzeria to your cluster creates a Kubernets secret named binding-conversation-service-watson-pizzeria.

* The secret contains a key named binding with its data being a JSON string of the form 

```
{"url":"https://gateway.watsonplatform.net/conversation/api",
 "username":"service-instance-user-uuid",
 "password":"service-instance-password"}.
```

The secret is mapped into the container as the environment variable CONVERSATION_SERVICE_WATSON_PIZZERIA through the Kubernetes pod configuration.

```
          - name: CONVERSATION_SERVICE_WATSON_PIZZERIA
            valueFrom:
              secretKeyRef:
                name: binding-conversation-service-watson-pizzeria
                key: binding
```

* The watson-conversation-slots-intro expects the service credentials to be set in the environment variables CONVERSATION_USERNAME and CONVERSATION_PASSWORD.

* We set these in the container environment using the run-server.sh script when the container is started. In this sample we parse them from the CONVERSATION_SERVICE_WATSON_PIZZERIA environment variable using the jq utility:

```
export CONVERSATION_USERNAME=$(echo "$CONVERSATION_SERVICE_WATSON_PIZZERIA" |
                                jq -r '.username')

```

You can see the defined secrets in the Kubernetes dashboard by running ``kubctrl proxy`` and accessing http://127.0.0.1:8001/ui
