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


#### Step 2: Create the Pod YAML File

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

#### Step 3: Scan the YAML File Using Kubesec

Once you have the YAML file ready, use Kubesec to scan the file for security vulnerabilities:

```bash
kubesec scan my-pod.yaml
```

Kubesec will return a detailed security report with a **score** for the Pod's security settings. Here’s what the output might look like:

### Results from First Scan (`my-pod.yaml`)

Here are the results from the first scan:

```json
{
  "object": "Pod/kubesec-demo.default",
  "valid": true,
  "fileName": "my-pod.yaml",
  "message": "Failed with a score of -29 points",
  "score": -29,
  "scoring": {
    "critical": [
      {
        "id": "Privileged",
        "selector": "containers[] .securityContext .privileged == true",
        "reason": "Privileged containers can allow almost completely unrestricted host access",
        "points": -30
      }
    ],
    ...
  }
}
```

### Interpretation of the Results

- **Score**: The score is **-29** points, indicating serious vulnerabilities.
- **Critical Issues**: 
  - **Privileged**: The Pod is running with a privileged context, which poses significant security risks. This can lead to unrestricted access to the host system.

- **Passed Checks**:
  - **ReadOnlyRootFilesystem**: The root filesystem is set to read-only, which is a good practice to prevent unauthorized modifications.

#### Step 4: Fix the Security Issue (Remove Privilege Escalation)

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

### Results from Second Scan (`my-pod2.yaml`)

Here are the results from the second scan:

```json
{
  "object": "Pod/kubesec-demo.default",
  "valid": true,
  "fileName": "my-pod2.yaml",
  "message": "Passed with a score of 1 points",
  "score": 1,
  "scoring": {
    "passed": [
      {
        "id": "ReadOnlyRootFilesystem",
        "selector": "containers[] .securityContext .readOnlyRootFilesystem == true",
        "reason": "An immutable root filesystem can prevent malicious binaries being added to PATH and increase attack cost",
        "points": 1
      }
    ],
    ...
  }
}
```

### Interpretation of the Second Scan Results

- **Score**: The score is **1** point, indicating a pass on the read-only filesystem check.
- **Passed Checks**: 
  - The only check passed is for the **ReadOnlyRootFilesystem**.

### Advise for Further Enhancements

While the Pod now has a score of 1, you should consider implementing additional security measures, such as:

- **AppArmor**: Define AppArmor policies for your containers to add another layer of security.
- **Service Accounts**: Use service accounts with the least privilege necessary to restrict Kubernetes API access.
- **Seccomp**: Implement Seccomp profiles to enforce security boundaries for system calls.
- **Resource Limits**: Specify resource limits and requests to prevent Denial of Service (DoS) attacks.


#### Step 6: Additional Use Cases for Kubesec

Kubesec can be applied beyond just scanning Pods. You can use it for:
- **Deployment configurations**: Scanning `Deployment` and `StatefulSet` YAML files to ensure secure configurations for multi-pod deployments.
- **Network Policies**: Ensuring that network policies are restrictive and do not allow unwanted traffic between services.
- **Role-based Access Control (RBAC)**: Evaluating the security of `Role` and `ClusterRole` configurations.

Kubesec's security scanning can be integrated into your CI/CD pipelines to automatically catch security misconfigurations before deploying to production.

### Conclusion

In this guide, we used Kubesec to scan a Kubernetes Pod for security risks, interpreted its findings, and applied security improvements to achieve a more secure setup. By integrating Kubesec scans into your Kubernetes workflow, you can ensure that your deployments adhere to best security practices from the start.





