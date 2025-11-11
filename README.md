
### Prerequisite setup

https://github.com/ajay-raut/k8s-postgres

---

(optional)Info regarding image used:
https://hub.docker.com/r/ajay25/go-gin-api

---
Here is a set of Kubernetes manifests to deploy your Go application and connect it to your existing PostgreSQL database.

This file creates two new resources:

1.  **A `Deployment`** for your `go-gin-api` application.
2.  **A `Service`** (using `NodePort`) to expose your API, similar to how you exposed Adminer.


### How to Apply

1.  Apply it to your `database-stack` namespace using this command:
    ```bash
    kubectl apply -f go-api-stack.yaml -n database-stack
    ```

### How to Access

To find the port your API is running on, run:
`kubectl get service -n database-stack`

You will see an output similar to this:

```
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
adminer-service    NodePort    10.100.123.45   <none>        8080:30007/TCP   10m
go-api-service     NodePort    10.100.56.78    <none>        8080:30008/TCP   1m
postgres-service   ClusterIP   10.100.99.123   <none>        5432/TCP         10m
```

You can then access your Go API at `http://<publicIP of worker>:<nodePort>` (e.g., `http://192.168.1.10:30008`).

-----

### **Important Assumptions**

I made a few key assumptions based on your files and common practices. **If your app doesn't start, check these first:**

1.  **Namespace:** I've placed your Go app in the **`database-stack`** namespace. This is **required** so it can find the database service at the hostname `postgres-service`.
2.  **Application Port:** I've assumed your Go Gin app listens on port **`8080`** inside the container (a very common default for Gin). If your code uses a different port (e.g., `r.Run(":8080")`), you **must** change `containerPort: 8080` in the Deployment and `targetPort: 8080` in the Service to match it.
3.  **Environment Variables:** I've assumed your Go code looks for database credentials using the environment variables `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, and `DB_PORT`. If your code uses different names (like `POSTGRES_HOST`), you must change the `name:` fields in the `env:` section of the `go-api-deployment` manifest.

---

You can check the status of your application using a few `kubectl` commands. Run these in your terminal.

Since you deployed everything to the `database-stack` namespace, be sure to add `-n database-stack` to each command.

### 1\. Check the Deployment

First, see if the deployment was successful and how many pods are "ready".

```bash
kubectl get deployment go-api-deployment -n database-stack
```

**✅ Good Output:**
You want to see the `READY` column match the `UP-TO-DATE` and `AVAILABLE` columns.

```
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
go-api-deployment   1/1     1            1           2m
```

-----

### 2\. Check the Pods

This is the most important step. It tells you if the container itself is running.

```bash
kubectl get pods -n database-stack
```

**✅ Good Output:**
Look for your `go-api-deployment-` pod and ensure its `STATUS` is **`Running`**.

```
NAME                                   READY   STATUS    RESTARTS   AGE
adminer-deployment-57...               1/1     Running   0          45m
go-api-deployment-6c...                1/1     Running   0          2m
postgres-deployment-7f...              1/1     Running   0          45m
```

**❌ Bad Output (Example):**
If you see `CrashLoopBackOff`, `Error`, or `Pending`, it means there's a problem.

```
NAME                                   READY   STATUS             RESTARTS   AGE
go-api-deployment-6c...                0/1     CrashLoopBackOff   3          2m
```

-----

### 3\. Check the Logs (If Pod is Not Running)

If the pod is not `Running` (e.g., `CrashLoopBackOff`), the best way to debug is to read its logs.

1.  First, get the exact pod name from the command above (e.g., `go-api-deployment-6c...`).
2.  Then, use that name to fetch its logs:

<!-- end list -->

```bash
# Replace <your-pod-name> with the name from the 'get pods' command
kubectl logs <your-pod-name> -n database-stack
```

  * **If the app can't connect to Postgres,** you will see the error message from your Go code here (e.g., "failed to connect to host=postgres-service").
  * **If the environment variables are wrong,** you might see an error like "database password not provided."

To watch the logs live (e.g., as you try to access the API), run:

```bash
kubectl logs -f <your-pod-name> -n database-stack
```

-----

### 4\. Check the Service

Finally, check that your `NodePort` service is up and has a port assigned.

```bash
kubectl get service go-api-service -n database-stack
```

**✅ Good Output:**
This shows the service is active and tells you which port to use (in this example, `30008`).

```
NAME             TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
go-api-service   NodePort   10.100.56.78    <none>        8080:30008/TCP   2m
```

You can then access your API at `http://<publicIP of worker>:30008`.

### 5\. For testing using Postman

Post method `http://<worker-publicIP>:30008/posts`

```
{
    "Title": "string2",
	"Body": "string3",
	"Author": "string333"
}

```

Put method `http://<worker-publicIP>:30008/posts/<id>`

```
{
    "Title": "300string"
}

```

Get (post by id) method `http://<worker-publicIP>:30008/posts/<id>`

Get (all post) method `http://<worker-publicIP>:30008/posts`

Delete (Delete post by id) method `http://<worker-publicIP>:30008/posts/<id>`


