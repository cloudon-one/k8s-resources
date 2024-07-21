# Istio Service Mesh and Envoy

## **I. Introduction :**

Deploying isn’t just about transferring your local work to a remote server; it’s more than just shifting environments. Deployment involves various layers and aspects. As data engineers, it’s crucial for us to gain expertise in these layers to ensure our contributions to the data science team stay reliable and relevant.

In today’s article, we’ll delve into Istio service mesh and its significance in microservice architecture.

## **II. What is a an Istio service mesh ?**

A service mesh is a tool integrated into your infrastructure, managing communication among services that need to interact. What makes it special? Well, it centralizes the communication process, eliminating the need to implement the business logic of communication or deal with associated overhead. In simpler terms, you won’t have to clutter your app with code to handle networking issues.

![https://miro.medium.com/v2/resize:fit:1362/1*LndO9veztu-eepOcjfmDmA.jpeg](https://miro.medium.com/v2/resize:fit:1362/1*LndO9veztu-eepOcjfmDmA.jpeg)

As depicted in the picture above, the application is split into multiple services collaborating to achieve its business objectives. We can observe the links between these services. As the application grows, managing this complexity becomes increasingly challenging. This is where the service mesh comes into play to address the issue. Now, let’s delve into discussing the role and components of a service mesh.

- **Proxy (Sidecar):** Functioning as a genuine router, its primary role is to direct requests from one service to another. It ensures secure communication between services in a cluster through TLS encryption, identity-based authentication, and authorization. Additionally, it collects metrics from various services, offering a comprehensive view of your application’s health.
- **Data Plane and Control Plane:** The data plane encompasses the sidecar proxies and manages the real network traffic. On the other hand, the control plane is responsible for configuring and overseeing the proxies to enforce desired behaviors and policies.

![https://miro.medium.com/v2/resize:fit:924/1*jE-dbJKAZteJKEC3IVGbgw.jpeg](https://miro.medium.com/v2/resize:fit:924/1*jE-dbJKAZteJKEC3IVGbgw.jpeg)

## **III. How does an Istio work ?**

Absolutely, keep in mind that the service mesh is logically divided into two key components: the data plane and the control plane.

The data plane consists of rules or deployed proxies that regulate communication between microservices in the application. Additionally, it takes on the responsibility of fetching metrics from various services.

On the other hand, the control plane manages and configures the proxies to direct traffic effectively. These two components work in tandem to orchestrate the communication and traffic flow within the service mesh.

## **IV. tutorial :**

In today’s tutorial, let’s imagine we have an e-commerce website comprising four microservices: the product page, reviews, ratings, and details. Your team has successfully completed the work on the website, and now it’s time for deployment. You’ve chosen to deploy it in a Kubernetes cluster, as illustrated in the picture below.

![https://miro.medium.com/v2/resize:fit:1400/1*H02_VPJqYQvr-slstX9uSw.jpeg](https://miro.medium.com/v2/resize:fit:1400/1*H02_VPJqYQvr-slstX9uSw.jpeg)

Our objective is to deploy the application, enable external accessibility through an ingress gateway, and establish communication between the microservices .

**Requirements :**

- [**minikube**](https://minikube.sigs.k8s.io/docs/start/)
- [**istioctl**](https://istio.io/latest/docs/reference/commands/istioctl/)

**step 1: install istioctl in your local machine**

Once Istioctl is installed, you need to configure the demo profile for our scenario by executing the following command in the terminal:

```
istioctl install --set profile=demo -y
```

![https://miro.medium.com/v2/resize:fit:1400/1*BjSCnN7svjOioK2ljHj_Xw.png](https://miro.medium.com/v2/resize:fit:1400/1*BjSCnN7svjOioK2ljHj_Xw.png)

**step 2 : start the minikube cluster**

```
minikube start
```

![https://miro.medium.com/v2/resize:fit:1140/1*MBgIDhSwy_3bjsswCiZRdg.png](https://miro.medium.com/v2/resize:fit:1140/1*MBgIDhSwy_3bjsswCiZRdg.png)

```
kubectl get pods -A
```

![https://miro.medium.com/v2/resize:fit:1142/1*hxs3Bd-DXlH2eYUM2iVkbw.png](https://miro.medium.com/v2/resize:fit:1142/1*hxs3Bd-DXlH2eYUM2iVkbw.png)

**Step 3: Add a namespace label to instruct Istio to automatically inject sidecar proxies when we deploy our application.**

```
 kubectl label namespace default istio-injection=enabled
```

![https://miro.medium.com/v2/resize:fit:1400/1*EtW-erfBq1D55Aa5Squ_OA.png](https://miro.medium.com/v2/resize:fit:1400/1*EtW-erfBq1D55Aa5Squ_OA.png)

**step 4: deploy the microservices on kubernetes**

Now, let’s navigate to the folder and initiate the deployment of the architecture mentioned above. If you’re not familiar with Kubernetes, no worries — I’ll provide explanations:

- **Deployment:** A deployment is a resource object that simplifies the scaling of containerized applications, ensuring that the specified desired state in the configuration is maintained.
- **Service:** Think of a service as a stable address that facilitates communication between different components of an application.
- **Service Account:** It serves as an identity that applications or processes inside a pod can use to interact with the Kubernetes API or other resources within the cluster.

we’ll execute these commands to initiate the deployment of the microservices .

**service accounts :**

```
 kubectl apply -f serviceAccountBookDetail.yaml
 kubectl apply -f serviceAccountRating.yaml
 kubectl apply -f serviceAccountReviews.yaml
 kubectl apply -f serviceAccountProductPage.yaml
```

![https://miro.medium.com/v2/resize:fit:1400/1*9T-RizUAQX-YO5nMqT4wSg.png](https://miro.medium.com/v2/resize:fit:1400/1*9T-RizUAQX-YO5nMqT4wSg.png)

**services :**

```
 kubectl apply -f serviceBookDetail.yaml
 kubectl apply -f serviceRating.yaml
 kubectl apply -f serviceReviews.yaml
 kubectl apply -f serviceProduct.yaml
```

![https://miro.medium.com/v2/resize:fit:1318/1*tsu-JJxOsibhtamtWJlP9w.png](https://miro.medium.com/v2/resize:fit:1318/1*tsu-JJxOsibhtamtWJlP9w.png)

**deployments :**

```
 kubectl apply -f deploymentBookDetail.yaml
 kubectl apply -f deploymentRating.yaml
 kubectl apply -f deploymentReviews.yaml
 kubectl apply -f deploymentReviewsV2.yaml
 kubectl apply -f deploymentReviewsV3.yaml
 kubectl apply -f deploymentProductPage.yaml
```

![https://miro.medium.com/v2/resize:fit:1400/1*IhO800r1zrC75LS4a2tMOQ.png](https://miro.medium.com/v2/resize:fit:1400/1*IhO800r1zrC75LS4a2tMOQ.png)

Now, the architecture has been successfully deployed.

![https://miro.medium.com/v2/resize:fit:1044/1*DoQ8fUyYv-uipQFNI4VjqA.png](https://miro.medium.com/v2/resize:fit:1044/1*DoQ8fUyYv-uipQFNI4VjqA.png)

Let’s run a command to test and check if we receive a response from the pod with the frontend interface.

```
kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```

![https://miro.medium.com/v2/resize:fit:1400/1*jeQZclSlCB7mnpzMWnq3tg.png](https://miro.medium.com/v2/resize:fit:1400/1*jeQZclSlCB7mnpzMWnq3tg.png)

As evident, we’ve successfully received the ‘Simple Bookstore App.

Currently, the application is only accessible within the Kubernetes cluster since all the services are of type ClusterIP. External requests cannot be sent. This is where the Ingress Gateway becomes crucial to facilitate external access.

**step 5: Create the Gateway to the application**

```
kubectl apply -f gateway.yaml
```

![https://miro.medium.com/v2/resize:fit:1302/1*9DmAJp85lw2yHjQNvLJH5Q.png](https://miro.medium.com/v2/resize:fit:1302/1*9DmAJp85lw2yHjQNvLJH5Q.png)

Let’s verify that the application is now accessible from our machine.

![https://miro.medium.com/v2/resize:fit:2000/1*tFhmBfRQmgxVMr4hI7-fUw.png](https://miro.medium.com/v2/resize:fit:2000/1*tFhmBfRQmgxVMr4hI7-fUw.png)

What we’ve accomplished so far is impressive. Now, let’s delve into an equally important aspect — visualizing and monitoring the microservices. Istio is capable of integrating with telemetry applications, making it crucial to briefly discuss these tools before advancing in the tutorial.

Absolutely, working with a microservices architecture introduces challenges in monitoring and performance management. Exploring tools tailored for these distributed environments becomes crucial to ensure effective service management and optimization.

1. **JAEGER**

![https://miro.medium.com/v2/resize:fit:1400/0*Mpno5tl3pkWZNSuo.png](https://miro.medium.com/v2/resize:fit:1400/0*Mpno5tl3pkWZNSuo.png)

Jaeger tracing is an open-source distributed tracing system used for monitoring microservices-based architectures. It provides insights into the flow and performance of requests across different services, aiding in issue identification and resolution. Jaeger enables detailed tracking of request lifecycles, helping developers optimize system behavior and improve overall reliability in distributed systems.

**2. Kiali**

![https://miro.medium.com/v2/1*YishSzZFrWdMIJS4JF6LZw@2x.png](https://miro.medium.com/v2/1*YishSzZFrWdMIJS4JF6LZw@2x.png)

Kiali is a user-friendly open-source dashboard designed for Istio, offering real-time insights into microservices. It simplifies the visualization of service interactions, aiding in the monitoring and management of complex microservices architectures for developers and operators.

In our tutorial, we've chosen Kiali to visualize our microservices architecture.

**step 6 : deploy Kiali dashboard**

```
kubectl apply -f monitoring
kubectl rollout status deployment/kiali -n istio-system
```

Accessing the Kiali dashboard from our local machine:

```
istioctl dashboard kiali
```

![https://miro.medium.com/v2/resize:fit:2000/1*qX0eZgJ2EWFW0AzrE0q6dA.png](https://miro.medium.com/v2/resize:fit:2000/1*qX0eZgJ2EWFW0AzrE0q6dA.png)

**Now, let’s analyze the dashboard:**

The Kiali dashboard presents an overview of our service mesh, depicting the relationships between services. It offers filtering options and visualizations for traffic flow. In the application section, we can observe the various microservices comprising our application.

![https://miro.medium.com/v2/resize:fit:2000/1*D3sm_Vi826XLalXQNaIWWQ.png](https://miro.medium.com/v2/resize:fit:2000/1*D3sm_Vi826XLalXQNaIWWQ.png)

In the Istio config section, we can find all the resources we’ve defined to expose our application externally, namely **VirtualService** and **Gateway**.

![https://miro.medium.com/v2/resize:fit:2000/1*xwlHDVGMMHjM03Z7gk7FRQ.png](https://miro.medium.com/v2/resize:fit:2000/1*xwlHDVGMMHjM03Z7gk7FRQ.png)

Right now, we’ll send a request and observe the graph section. By sending a request, we can simply refresh the page of our product page, and let’s see what happens.

![https://miro.medium.com/v2/resize:fit:2000/1*HSIVHH8FN1WQIopsgC2w6A.png](https://miro.medium.com/v2/resize:fit:2000/1*HSIVHH8FN1WQIopsgC2w6A.png)

That’s truly impressive! The dashboard provides a clear visualization of the request flow. From the left, we can observe requests initiated from the outside, invoking the product microservice. This microservice then retrieves detailed information from the detail microservice and gathers reviews from the reviews microservice. The dashboard offers an excellent illustration, allowing us to trace request data and analyze the utilization and time taken by each microservice. Clicking on specific microservices on the graph enables us to further explore the time required to reach one microservice from another.

![https://miro.medium.com/v2/resize:fit:764/1*GiYnMolYmkBAK5flybVqIA.png](https://miro.medium.com/v2/resize:fit:764/1*GiYnMolYmkBAK5flybVqIA.png)

## **V. Conclusion :**

I genuinely enjoy crafting these articles and sharing my learning methods with you all. While it might not be the ultimate approach, I’m committed to continuous improvement, and your feedback is greatly appreciated. My goal is to inspire you to persist in your learning journey and always strive for excellence. If you have ideas or want to collaborate on something, feel free to reach out — I’m open to working together and making progress collectively.