# Kubernetes Manifests for Jenkins Deployment

Refer https://devopscube.com/setup-jenkins-on-kubernetes-cluster/ for step by step process to use these manifests.

The steps below (and associated files) are nearly identical to the steps as described by [devops cube](devopscube.com) in the link above. However, I have followed these steps with [Docker Desktop using the local Kubernetes cluster](https://docs.docker.com/desktop/kubernetes/). I hope this encourages you to try this for yourself and learn more about Kubernetes, Jenkins, Docker, and DevOps!

1. Create a Kubernetes **Namespace** for Jenkins.

    ```bash
        kubectl create namespace devops-tools
    ```

2. Create the *service account* (`serviceAccount.yaml`) using `kubectl`.

    ```bash
        kubectl apply -f serviceAccount.yaml
    ```

    * This YAML file does the following:
  
      1. Creates a `jenkins-admin` **ClusterRole**.

      2. Creates a `jenkins-admin` **ServiceAccount**.

      3. Binds the **ClusterRole** to the **ServiceAccount**.

3. Create a *persistent volume* (`volume.yaml`) in one of your worker nodes (`kubectl get nodes`) using `kubectl`.

    ```bash
        kubectl apply -f volume.yaml
    ```

    * This YAML does the following:

      1. Creates a `local-storage` **StorageClass**.

      2. Creates a `jenkins-pv-volume` **PersistentVolume**.

      3. Claims 3Gi for `jenkins-pv-claim` **PersistentVolumeClaim**.

    * When a *pod* is deleted, it's data will persist in the *node* volume. However, when a *node* is deleted, all the data will be lost.

4. Create the *deployment* (`deployment.yaml`) using `kubectl`.

    ```bash
        kubectl apply -f deployment.yaml
    ```

    * This YAML does the following:

      1. Creates a `jenkins` **Deployment**.

          1. Specifies a `securityContext` for the Jenkins pod to read and write to the **PersistentVolume**.

          2. Specifies a `readinessProbe` and `livenessProbe`.

          3. Specifies a local persistent volume that holds the Jenkins data path `/var/jenkins_home`.

    * Check the deployment status using:

        ```bash
            kubectl get deployments -n devops-tools
        ```

    * Get deployment details using:

        ```bash
            kubectl describe deployments --namespace=devops-tools
        ```

5. Create a *service* (`service.yaml`) using `kubectl`.

    ```bash
        kubectl apply -f service.yaml
    ```

    * This YAML does the following:

      1. Creates a `jenkins-service` **Service**.

          1. Deploys in `devops-tools` **Namespace**.

          2. Expose `jenkins-server` to *all* Kubernetes nodes on port 32000.

          3. Maps port 32000 to port 8080.

    * Get the service details using:

        ```bash
            kubectl get services -n devops-tools
        ```

    * Since this Jenkins server is deployed *locally* you can access the dashboard via `http://localhost:32000`. Follow the steps below to get the administrator password.

        1. Get the **Pod** name with:

            ```bash
                kubectl get pods -n devops-tools
            ```

        2. Use the **Pod** name obtained previously with:

            ```bash
                kubectl logs jenkins-85fcfbb869-mcrdp -n devops-tools
            ```

            * The administrator password can be parsed and printed to the terminal by passing a command to the service with:

                ```bash
                    kubectl exec -it jenkins-85fcfbb869-mcrdp cat /var/jenkins_home/secrets/initialAdminPassword -n devops-tools
                ```

        3. After entering the initial administrator password you can install the suggested plugins and create an admin user.
