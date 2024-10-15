### Securing-K8s-6: Using Kubesec to Scan and Improve Kubernetes Pod Security

In this guide, we will use **Kubesec**, a tool designed to scan Kubernetes resource definitions for security concerns, focusing on a specific use case: enhancing the security of a Kubernetes Pod YAML configuration. By leveraging Kubesec's scoring system, we can identify and address potential vulnerabilities in the pod’s configuration.

#### Step 1: Download and Install Kubesec

First, download Kubesec on your remote machine. Follow the commands below:

```bash
curl -LO https://github.com/controlplaneio/kubesec/releases/download/v2.14.1/kubesec_linux_amd64.tar.gz
tar -xzf kubesec_linux_amd64.tar.gz
chmod +x kubesec
sudo mv kubesec /usr/local/bin/kubesec
```

This will download the latest version of Kubesec, extract the binary, make it executable, and move it to your system’s path.

#### Step 2: Start the Kubesec HTTP Server

To scan YAML files using Kubesec's HTTP API, start the Kubesec HTTP server:

```bash
./kubesec http 8080 &
```

This starts the server in the background, making it accessible on port `8080`.

#### Step 3: Create the Pod YAML File

We will now define a Kubernetes Pod that we want to scan. Below is a YAML configuration file (as shown in the image). Save this configuration as `my-pod.yaml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubesec-demo
spec:
  containers:
    - name: kubesec-demo
      image: nginx
      securityContext:
        privileged: true
        readOnlyRootFilesystem: true
```

In this configuration, the Pod contains one container running the `nginx` image. It uses the `privileged` mode, which we will address later.

#### Step 4: Scan the YAML File Using Kubesec

Once you have the YAML file ready, use Kubesec to scan the file for security vulnerabilities:

```bash
kubesec scan my-pod.yaml
```

Kubesec will return a detailed security report with a **score** for the Pod's security settings. Here’s what the output might look like:

```
File: my-pod.yaml
Score: -3

Advise:
- Privilege escalation should be avoided unless absolutely necessary.
- Containers should not run as privileged.
- 'readOnlyRootFilesystem' should be true for extra security.
```

##### Understanding the Score:
- The **score** reflects how secure the Pod configuration is. A **positive score** indicates that the security posture is acceptable, while a **negative score** (as seen here) shows critical vulnerabilities.
- The **privileged: true** setting is the main issue here, as it allows the container to access the host system with elevated privileges, which poses a serious security risk.
- The setting `readOnlyRootFilesystem: true` is good practice and contributes positively to the security score by ensuring that the container’s filesystem cannot be written to.

#### Step 5: Fix the Security Issue (Remove Privilege Escalation)

To improve the security, we will remove the `privileged: true` setting from the `securityContext`. Modify your `my-pod.yaml` to look like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubesec-demo
spec:
  containers:
    - name: kubesec-demo
      image: nginx
      securityContext:
        readOnlyRootFilesystem: true
```

Now, re-run the Kubesec scan:

```bash
kubesec scan my-pod.yaml
```

You should see an improved score, with the issue of privilege escalation removed:

```
File: my-pod.yaml
Score: 5

Advise:
- 'readOnlyRootFilesystem' is set to true, providing extra security.
```

##### Analyzing the Results:
- **Improved score**: By removing the `privileged` flag, we have greatly improved the security of the Pod. 
- **Remaining suggestions**: Kubesec will still offer advice on other improvements, such as ensuring the container is running with the least privilege possible or tightening file permissions further.

#### Step 6: Additional Use Cases for Kubesec

Kubesec can be applied beyond just scanning Pods. You can use it for:
- **Deployment configurations**: Scanning `Deployment` and `StatefulSet` YAML files to ensure secure configurations for multi-pod deployments.
- **Network Policies**: Ensuring that network policies are restrictive and do not allow unwanted traffic between services.
- **Role-based Access Control (RBAC)**: Evaluating the security of `Role` and `ClusterRole` configurations.

Kubesec's security scanning can be integrated into your CI/CD pipelines to automatically catch security misconfigurations before deploying to production.

### Conclusion

In this guide, we used Kubesec to scan a Kubernetes Pod for security risks, interpreted its findings, and applied security improvements to achieve a more secure setup. By integrating Kubesec scans into your Kubernetes workflow, you can ensure that your deployments adhere to best security practices from the start.
