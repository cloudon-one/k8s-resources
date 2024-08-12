# GitOps Secrets with Argo CD, Hashicorp Vault and the External Secret Operator

Teams adopting GitOps often ask how to use secrets with Argo CD. The official Argo CD page about secrets [is unopinionated by design](https://argo-cd.readthedocs.io/en/stable/operator-manual/secret-management/) and simply lists a set of projects that can help you with secrets.

We’ve seen several approaches to secret management. These include [sealed secrets](https://codefresh.io/blog/handle-secrets-like-pro-using-gitops/), the [Argo CD Vault plugin](https://codefresh.io/blog/aws-secret-manager-argocd/), and the [External Secret Operator](https://codefresh.io/blog/aws-external-secret-operator-argocd/).

In this post, we showcase the [External Secret Operator](https://external-secrets.io/latest/) and [Hashicorp Vault](https://www.vaultproject.io/)and focus on 2 important aspects.

- How to avoid saving ANY secrets in Git, including tokens for fetching the application secrets
- How to refresh secrets automatically without pod restarts and application deployments

Several existing tutorials show how you don’t need to store any application secrets in Git, but never actually explain where to store the token for fetching the secrets from Vault or other secret providers. So you still have the same problem but for the token itself instead of the application secrets — where to store it?

Refreshing secrets automatically is also fundamental, as passing secrets to an application is only half the battle. You also need an easy way to rotate and revoke compromised secret information.

# **How to pass secrets with the External Secret Operator**

The External Secret Operator (ESO) is a Kubernetes controller that supports retrieving secrets from several providers (AWS, Azure, GCP, Vault, Gitlab, etc) and converting them to plain Kubernetes secrets so any Kubernetes application can consume them.

![https://miro.medium.com/v2/resize:fit:1400/0*ZeSarjfK5WeBXhQo.png](https://miro.medium.com/v2/resize:fit:1400/0*ZeSarjfK5WeBXhQo.png)

The Controller is designed for Kubernetes installations and although it works great with Argo CD, it doesn’t need Argo CD or even follow [GitOps principles](https://codefresh.io/learn/gitops/). It introduces a set of custom resources (CRDs) for representing a secret source, SecretStore, and a secret from that store, ExternalSecret.

One of the best characteristics of ESO is that all secrets it controls get converted into plain [Kubernetes secrets](https://kubernetes.io/docs/concepts/configuration/secret/) in the end (regardless of their actual source). This makes it a very flexible choice because the application code can be as minimal as possible.

In practice, your application code:

- Doesn’t know anything about the ESO controller
- Doesn’t need a special API to access secrets
- Doesn’t even need to know it’s running inside Kubernetes
- Can read secret values from files (recommended) or environment variables

This is one of the controller’s highlight features, especially in legacy applications where you can’t change the source code at all. ESO is also a great choice when you want to move secrets from one provider to another, as the application is completely oblivious to where the secrets come from.

# **The example secret application**

Our demo application is at [https://github.com/kostis-codefresh/external-secrets-gitops-example/tree/main/src](https://github.com/kostis-codefresh/external-secrets-gitops-example/tree/main/src).

It’s a very simple application that reads secrets from files mounted at /secrets.

![https://miro.medium.com/v2/resize:fit:1400/0*9-LCf1rUHEJmUVeQ.png](https://miro.medium.com/v2/resize:fit:1400/0*9-LCf1rUHEJmUVeQ.png)

We recommend reading secrets from files and not environment variables, as you can monitor files for changes. When you need to rotate or revoke secrets, it would be great to make your application consume the new secrets without any restarts or redeployments. Of course, you can also use Kubernetes [secrets as environment variables](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data).

The application needs 3 secrets:

1. Database URL
2. Database username
3. Database password

Just for demo purposes, there’s no real database. The application simply prints the secrets so you can verify visually if the values in Hashicorp Vault have passed successfully to the pod.

![https://miro.medium.com/v2/resize:fit:1400/0*BagTv9dDMBSkdV51.png](https://miro.medium.com/v2/resize:fit:1400/0*BagTv9dDMBSkdV51.png)

Notice the application clearly says where these secrets get loaded from (/secrets directory). This is a great practice that lets you know the source of your secrets, ensuring you don’t miss any during rotation. You can expose this using an API instead, or even have an organization-wide convention that specifies the source of secrets. We recommend you do this regardless of your adoption of Argo CD or GitOps.

When trying to understand how an application loads configuration and secrets, having the application tell you itself saves time spent debugging and looking at logs. This is especially helpful during an incident where time is of the essence.

# **Installing Hashicorp Vault and ESO in your Kubernetes cluster**

Let’s start first by deploying Hashicorp Vault and ESO in your cluster. You can do it with Argo CD and simply use the public Helm charts.

![https://miro.medium.com/v2/resize:fit:1400/0*sC4qRO8GcymjBuRH.png](https://miro.medium.com/v2/resize:fit:1400/0*sC4qRO8GcymjBuRH.png)

In a real company, Hashicorp Vault should be already installed somewhere else by your administrator. You’d need to ask your security team for access.

For this demo, we installed Hashicorp Vault with the **server.dev.enabled**option as **true**, so that Vault runs in development mode and doesn’t need [sealing/unsealing](https://developer.hashicorp.com/vault/docs/concepts/seal). **This is only for demo purposes**. The default root token is also set as “root” and you can use it to log in to the UI as well.

# **Fetching external secrets from Hashicorp Vault**

Before we pass secrets to our application we need to explain to ESO where to get them. In our example, we use the [built-in integration for Hashicorp Vault](https://external-secrets.io/latest/provider/hashicorp-vault/). For the integration to work, we need to create a SecretStore or ClusterSecretStore resource that defines how to authenticate against Vault to get secrets.

This is the crucial point of the whole process. Several existing tutorials about external secrets either don’t explain the integration process, or use a predefined token for accessing Vault (or any other secret provider).

If you use an authentication token, you’re introducing the chicken-and-egg question about how to store the token itself. Using external secrets avoids storing the secrets in Git. But if we need a token for accessing secrets, where should we store that token? If we choose Git, we’re back to square one. If we hard-code in a manifest and push it on the cluster, we’ve introduced a single point of failure for reliability and security reasons.

The answer to this question is provider-specific, but essentially you need to use an authentication method based on trust and not static tokens. Several ESO providers support authentication methods with this characteristic. You need to contact your security team and discuss your options with them. Make sure to check the support of the Open ID Connect protocol (OIDC) for your secret provider.

![https://miro.medium.com/v2/resize:fit:1400/0*sJi6KGceWtXBCl4V.png](https://miro.medium.com/v2/resize:fit:1400/0*sJi6KGceWtXBCl4V.png)

This guide isn’t about Hashicorp Vault, but since we’re using it as an example, we can easily follow this best practice and use a trust method instead of a predefined token.

For our demo, we can instruct Hashicorp Vault to [use Kubernetes authentication](https://external-secrets.io/latest/provider/hashicorp-vault/#kubernetes-authentication) by trusting the same Kubernetes cluster that ESO is running on. This way we don’t need to store any token at all. ESO will use a Kubernetes service account from the pod it’s running on and pass that to Vault. Vault has many more authentication options, like trusting outside Kubernetes clusters or even other Vault instances, but explaining them is outside this guide’s scope.

The result is that we didn’t hardcode any secrets or tokens anywhere, in the cluster or in Git.

To enable the [Kubernetes authentication read the Vault documentation](https://developer.hashicorp.com/vault/docs/auth/kubernetes). You can use the UI or the CLI to enable it. In our example cluster, you can expose the Vault UI with:

```
kubectl port-forward -n vault vault-0 8200:8200
```

The default token is “root” because we used the “dev” installation mode.

![https://miro.medium.com/v2/resize:fit:1400/0*lRcHVyjZUBmSAanE.png](https://miro.medium.com/v2/resize:fit:1400/0*lRcHVyjZUBmSAanE.png)

A nice part of the default installation of Vault in Kubernetes, is that you get the Vault CLI preinstalled as well in the same pod. So enabling Kubernetes auth for a quick demo is as simple as doing the following:

```
kubectl exec --stdin=true --tty=true -n vault vault-0 -- /bin/sh
vault auth enable kubernetes
vault write auth/kubernetes/config \
  kubernetes_host=https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT

vault write auth/kubernetes/role/demo \
  bound_service_account_names=* \
  bound_service_account_namespaces=* \
  policies=default \
  ttl=1h
```

You also need to change the default policy to allow reading of secrets by adding the following snippet to the default policy:

```
 path "secret/*" {
  capabilities = [ "read", "list" ]
}
```

The rest of the policy should stay as-is.

![https://miro.medium.com/v2/resize:fit:1400/0*T8kRseW1doPb3-5n.png](https://miro.medium.com/v2/resize:fit:1400/0*T8kRseW1doPb3-5n.png)

Your dedicated security team should handle these settings for you in Vault or your preferred secret providers. The settings above are just for demo purposes to show you how external secrets work.

The last step is creating a [ClusterSecretStore](https://github.com/kostis-codefresh/external-secrets-gitops-example/blob/main/manifests/vault-integration/secretstore.yml) that tells ESO that Vault will be used for secret storage.

```
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "http://vault.vault:8200"
      path: "secret"
      # Version is the Vault KV secret engine version.
      # This can be either "v1" or "v2", defaults to "v2"
      version: "v2"
      auth:
        # points to a secret that contains a vault token
        # https://www.vaultproject.io/docs/auth/token
        kubernetes:
          mountPath: "kubernetes"
          role: "demo"
```

Notice there are no credentials in this file, so it can be safely stored in Git as-is.

# **Passing secrets from Vault to your application**

With the setup done, we’re now ready to use secrets in our application. First, create a set of secrets in Vault. You can do this with the CLI, the UI, Terraform, etc.

![https://miro.medium.com/v2/resize:fit:1400/0*ooTDKZ9NNHUFREXW.png](https://miro.medium.com/v2/resize:fit:1400/0*ooTDKZ9NNHUFREXW.png)

Next, create an [External Secret YAML](https://github.com/kostis-codefresh/external-secrets-gitops-example/blob/main/manifests/app/external-secret-db.yml).

```
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-db-credentials
spec:
  refreshInterval: "15s"
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: mysql-credentials
    template:
      engineVersion: v2
      data:
        credentials: |
          db_con="{{ .db_url }}"
          db_user="{{ .db_username }}"
          db_password="{{ .db_password }}"

  data:
  - secretKey: db_url
    remoteRef:
      key: mysql_credentials
      property: url
  - secretKey: db_username
    remoteRef:
      key: mysql_credentials
      property: username
  - secretKey: db_password
    remoteRef:
      key: mysql_credentials
      property: password
```

In this file, we’re using [secret templating](https://external-secrets.io/latest/guides/templating/) to match the file format that the application expects (a simple [property file](https://github.com/kostis-codefresh/external-secrets-gitops-example/blob/main/src/credentials)). Again, notice this file doesn’t have any confidential information. It’s simply a pointer to Hashicorp Vault. This means you can safely store it in Git (and apply it with Argo CD).

Finally, deploy the application with Argo CD. Here’s the [deployment file](https://github.com/kostis-codefresh/external-secrets-gitops-example/blob/main/manifests/app/deployment.yml).

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitops-secrets-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gitops-secrets-app
  template:
    metadata:
      labels:
        app: gitops-secrets-app
    spec:
      containers:
      - name: gitops-secrets-app
        image: docker.io/kostiscodefresh/simple-secret-app:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: mysql
          mountPath: "/secrets"
          readOnly: true
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
      volumes:
      - name: mysql
        secret:
          secretName: mysql-credentials
```

Notice that it mounts the mysql-credentials secret as a normal file in the container filesystem at the /secrets folder.

Now launch the application and you can see it has the correct secret information. You can do that with:

```
kubectl port-forward svc/gitops-secrets-service 8080:8080
```

Success! Our secrets from Vault are accessible to our application without saving anything in Git.

![https://miro.medium.com/v2/resize:fit:1400/0*Ag_Qv6iUg9gpivcp.png](https://miro.medium.com/v2/resize:fit:1400/0*Ag_Qv6iUg9gpivcp.png)

If we look at the Resource Overview in Argo CD we can see that ESO automatically generated a standard secret for us with Vault information. That secret was mounted on the container.

![https://miro.medium.com/v2/resize:fit:1400/0*QpfDW5tkZxuS15rh.png](https://miro.medium.com/v2/resize:fit:1400/0*QpfDW5tkZxuS15rh.png)

Here’s what happened behind the scenes:

1. We applied our [application manifests](https://github.com/kostis-codefresh/external-secrets-gitops-example/tree/main/manifests/app) with Argo CD.
2. The service and deployment are standard Kubernetes resources and were applied directly by Argo CD.
3. The external secret is a CRD. It was passed to the external secret controller.
4. ESO sees that this is an external secret that mentions our Vault secret store.
5. ESO contacts Vault and asks for secret details.
6. Vault trusts ESO because Kubernetes authentication is active and they are running on the same Kubernetes cluster.
7. Vault gives the secret information to ESO.
8. ESO creates a standard Kubernetes secret using the template defined in the ExternalSecret CRD.
9. The job of ESO is over.
10. A standard Kubernetes deployment mounts the Kubernetes secret at /secrets/credentials in the filesystem of the container.
11. The application starts up. It reads /secrets/credentials like any other file without any knowledge of how this file was created.
12. The secrets appear on the web page.

# **Refreshing secrets without application restarts**

In the introduction, we mentioned that using secrets is just one part of the equation. We also need a way to easily rotate and revoke secrets.

One of the highlight features of ESO is that it can automatically refresh our secrets when they change in the original secret storage.

Our application takes advantage of this automatically.

1. It loads the secret from a file and not an environment variable. Environment variables are fixed once a process starts and the only way to refresh them is to restart the app.
2. The source code [automatically monitors the /secrets/credentials](https://github.com/kostis-codefresh/external-secrets-gitops-example/blob/main/src/simple-web-server.go#L85) file for changes and auto-reloads it if it changes.

This means that we can change our secrets on the fly and see the application update right away. Let’s say that our database password became compromised and we had to update it. We’d make the change in Vault:

![https://miro.medium.com/v2/resize:fit:1400/0*iG-b1u3y9FcSqIMS.png](https://miro.medium.com/v2/resize:fit:1400/0*iG-b1u3y9FcSqIMS.png)

In the next refresh period (defined in the External Secret CRD with the refreshInterval property), ESO will pass the new change to the application.

The application will update the secret on its own. How nice is that? 🙂

![https://miro.medium.com/v2/resize:fit:1400/0*d6lhmZNDNphOOcwz.png](https://miro.medium.com/v2/resize:fit:1400/0*d6lhmZNDNphOOcwz.png)

We achieved one of the holy grails of good security practices. We can rotate and revoke secrets without restarting the application and without recreating any pods.

# **Conclusion**

In this guide, you saw an alternative method of using secrets with GitOps, where an external source keeps the secret values. In Git, we only stored their configuration and how they’re mounted on their application. If your organization already has an external source for secrets, check if the External Secret Operator supports it.

You also saw how to prepare an application to automatically reload secrets when they change, making secret rotation simple.

We hope that now you have several choices for managing secrets with GitOps, you can choose your own level of risk or ease of use based on how closely you want to follow GitOps principles.