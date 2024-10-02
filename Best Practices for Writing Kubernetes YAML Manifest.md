# Best Practices for Writing Kubernetes YAML Manifests

Kubernetes objects are deployed to Kubernetes clusters using configuration files written in YAML, often referred to as YAML manifests. You specify the “desired state” of objects in a manifest file and send the file to the Kubernetes API server. Kubernetes then automatically configures and manages the application based on your specifications. In this blog post, we’ll outline five best practices you should remember while writing Kubernetes YAML manifests. Let’s get started!‍

# **#1 Use the latest stable API version**

Kubernetes API versions typically go through three stages:

1. **Alpha:** The version names contain “alpha” (e.g., `v1alpha1`). These are experimental features that may be unstable and are disabled by default. Alpha APIs can change without notice and are not recommended for production use.
2. **Beta:** The version names contain “beta” (e.g., `v2beta3`). These are well-tested features, but are disabled by default. Beta features are considered safe to enable but are not recommended for production use as they may still undergo breaking changes.
3. **Stable:** The version names are simply “vX” where X is an integer (e.g., `v1`). These are production-ready features that are fully supported, enabled by default, and maintain backwards compatibility.‍

To leverage these versioned APIs, every Kubernetes object manifest must specify the API version in a field named `apiVersion`. This field tells Kubernetes which version of the API to use when processing the manifest.

The `apiVersion` field typically consists of two parts: the API group (such as `apps` or `batch`) and the actual version (such as `v1`). Note that for core Kubernetes objects, only the version is specified.

Here’s an example manifest for a Deployment object:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.27‍
```

In this example, `apiVersion`: `apps/v1` indicates that this Deployment object uses the `v1` version of the `apps` API group, which is the latest stable version for Deployments. When creating Kubernetes object manifests, you must always use the latest stable API version available for each object type.

Here’s why:

- **Reliability:** Stable APIs are less likely to introduce breaking changes, ensuring that your applications remain functional over time.
- **Support:** Stable versions receive regular updates and support from the Kubernetes community, making it easier to find help and resources.
- **Future-proofing:** By adopting stable APIs, you position your applications to benefit from ongoing enhancements and avoid the risks associated with deprecated or unstable versions.‍

To find out the latest stable API version for an object on your current Kubernetes cluster, you can use the `kubectl api-resources` command. This command queries the Kubernetes API server you're connected to and lists all available resources and their API versions supported by that specific cluster.

# **#2 Use labels that identify semantic attributes of your application**

In Kubernetes, labels are arbitrary key/value pairs that you can attach to objects. They provide a flexible way to organize and categorize objects and manage them efficiently. Labels are particularly useful when combined with label selectors, which allow you to filter and operate on specific sets of objects.‍

Consider this example Deployment manifest:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v1‍
```

While these labels are valid, they don’t fully capture the semantic attributes of the application. In other words, these labels don’t convey meaningful characteristics about the application. A better way to label the application would be to add more descriptive labels.

The Kubernetes official documentation recommends a set of [common labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/)that you can apply to your object manifest. These labels, which begin with the prefix `app.kubernetes.io/` followed by a separator (`/`), provide a standardized way to describe your application's components and improve interoperability with various Kubernetes tools and systems.‍

Here’s what an improved version of the aforementioned Deployment manifest could look like, incorporating these recommended labels:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app.kubernetes.io/name: myapp
    app.kubernetes.io/version: "1.0.0"
    app.kubernetes.io/component: frontend
    app.kubernetes.io/part-of: web-application
spec:
  replicas: 3
  selector:
    matchLabels:
      app.kubernetes.io/name: myapp
        template:
    metadata:
      labels:
        app.kubernetes.io/name: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v1.0‍
```

These semantic labels provide much more context about the application. They describe not just what the application is, but also its version and its role in the larger system.‍

The benefits of this semantic labelling approach include:

1. **Improved organization:** Resources are grouped in a more meaningful way.
2. **Enhanced querying:** You can easily find all frontend components or all resources related to a specific application.
3. **Clearer communication:** Team members can quickly understand the purpose and context of each resource.
4. **Interoperability:** The use of standardized `app.kubernetes.io/` labels enables different Kubernetes tools to work together seamlessly, recognizing and utilizing the same information across various platforms and systems.‍

By using semantic labels consistently across your Kubernetes objects, you create a more self-documenting system that is easier to understand and manage.‍

# **#3 Put object descriptions in annotations for better introspection**

In Kubernetes, annotations are key-value pairs that allow you to attach non-identifying metadata to objects. Typical examples of annotations include build information, release IDs, Git branch names, PR numbers, image hashes, registry information, or team contact details.‍

Annotations provide a way to examine and understand the objects in the cluster more deeply. This is what the phrase “better introspection” means in the context of Kubernetes annotations.

Here’s an example of a Pod with three annotations for commit, author, and branch:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  annotations:
    git.commit: "7a8b9c0d1e2f3g4h5i6j7k8l9m0n1o2p"
    git.author: "Sarah Chen <sarah.chen@example.com>"
    git.branch: "feature/custom-nginx-config"
spec:
  containers:
  - name: nginx
    image: nginx:1.27‍
```

In this example, annotations provide valuable context about the specific version of the NGINX configuration being deployed, which can be extremely useful for debugging, auditing, and managing your Kubernetes Deployments. While these annotations offer useful information for human readers, their power extends far beyond simple documentation.

In fact, annotations are primarily used to provide additional context or configuration information that can be utilized by external tools, automation systems, or client libraries interacting with the Kubernetes API.‍

For example:

- A CI/CD tool might use annotations to store information about the build process.
- A monitoring tool might use annotations to specify how to scrape metrics or which alerts to associate with a particular resource.
- A custom deployment tool might use annotations to store information about rollout strategies or canary deployments.‍

Therefore, always ensure to include descriptive and relevant annotations in your Kubernetes object definitions to enhance manageability and observability.‍

## **Key Differences in Allowed Characters**

- **Labels:** Keys are more restricted; both the prefix and the name must conform to DNS subdomain rules and specific character restrictions. Values must be relatively short (63 characters or less) and are limited to certain characters.
- **Annotations:** Keys follow the same rules as labels, but values have no strict size limits and can contain any UTF-8 characters, allowing for much more flexibility in what they can store.‍‍

# **#4 Don’t hardcode secret data**

Secret data consists of sensitive information that should be protected from unauthorized access. This includes passwords, API keys, tokens, and other confidential data. A common mistake in Kubernetes deployments is hardcoding this sensitive information directly into manifest files.‍

To illustrate this issue, let’s examine the following Deployment manifest:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql-container
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "my-secret-password"
```

In this manifest, we have hardcoded the value for the root password directly in the environment variable. This exposes the password in plain text, posing significant security risks.‍

A more secure approach is to store the root password in a Kubernetes Secret object and then reference it in the Deployment. Here’s how you can create a Secret:

```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-root-pass
type: Opaque
stringData:
  password: mysql_secret_password‍
```

Once you’ve created the Secret, you can reference it in your Deployment manifest by mounting it as an environment variable:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql-container
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-root-pass
              key: password‍
```

Using Secrets in this manner offers several key benefits:

1. **Separation of concerns:** Keeping sensitive data separate from application code adheres to best practices in configuration management.
2. **Easier management:** Secrets can be updated independently of the Deployment, allowing for easier rotation of credentials.
3. **Environment-specific configuration:** Different Secret objects can be created for various environments, facilitating consistent application behavior across different setups.‍

# **#5 Combine related Kubernetes objects into one file**

When working with Kubernetes, you often need to create multiple related objects to deploy an application. While it’s possible to define each object in a separate file, the following are some scenarios where combining related Kubernetes objects into a single file can be beneficial.‍

- For small to medium-sized applications where all components are tightly coupled.
- When you want to deploy an entire application stack with a single command.
- For objects that are always deployed together and have a strong relationship.‍

Let’s explore this concept with an example. Imagine you want to deploy a backend API server for a simple web application. Traditionally, you might have separate YAML files for this component: `backend-deployment.yaml` and `backend-service.yaml`.‍

## **backend-deployment.yaml**

```
apiVersion: v1
kind: Pod
metadata:
 name: nginx-web-server
 labels:
   app.kubernetes.io/name: nginx-web-server
spec:
 containers:
   - name: nginx-web-server
     image: nginx:1.27
     ports:
       - containerPort: 80
         name: http-web-svc‍
```

## **backend-service.yaml**

```
apiVersion: v1
kind: Service
metadata:
 name: nginx-web-server-service
spec:
 selector:
   app.kubernetes.io/name: nginx-web-server
 ports:
   - name: http
     protocol: TCP
     port: 80
     targetPort: http-web-svc‍
```

This approach results in two separate files that you need to manage and deploy individually.

Instead, you could combine the two objects into a single `backend.yaml` file. This f ile would contain the Deployment and the Service for your backend, separated by `---`delimiters, like this:

## **backend.yaml**

```
apiVersion: v1
kind: Pod
metadata:
 name: nginx-web-server
 labels:
   app.kubernetes.io/name: nginx-web-server
spec:
 containers:
   - name: nginx-web-server
     image: nginx:1.27
     ports:
       - containerPort: 80
         name: http-web-svc
---
apiVersion: v1
kind: Service
metadata:
 name: nginx-web-server-service
spec:
 selector:
   app.kubernetes.io/name: nginx-web-server
 ports:
   - name: http
     protocol: TCP
     port: 80
     targetPort: http-web-svc‍
```

This combined approach offers several benefits:

- **Simplified deployment:** You can deploy all objects with a single `kubectl apply -f backend.yaml` command.
- **Improved version control:** Changes to related objects are tracked together in your version control system.
- **Better readability:** It’s easier to understand the full application structure when all components are in one file.
- **Easier troubleshooting:** When all related objects are in one file, it’s simpler to identify and fix issues.‍

However, it’s important to note that this approach works best for smaller applications or tightly coupled components. For larger, more complex applications, you might still prefer to keep objects in separate files for better modularity and easier management of individual components.‍

# **Conclusion**

Adhering to best practices for writing Kubernetes YAML manifests is essential, particularly when collaborating within a team or managing complex applications. By consistently implementing these practices, you not only streamline your workflow but also enhance the clarity, security, and organization of your deployments. You contribute to a more efficient and collaborative development environment, which leads to improved productivity and a faster software development lifecycle.

Additionally, utilizing tools such as the [YAML extension](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) for Visual Studio Code can further improve your efficiency when working with YAML manifests. This extension provides features like auto-completion, error highlighting, and snippets, which can help you avoid syntax errors and speed up the file creation process, making it an invaluable tool for any Kubernetes user.