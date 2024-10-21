# k3d-rabbitmq-keda-tutorial
k3d-rabbitmq-keda-tutorial for learning how work messaging and auto-scale </br>


  <strong><h1>Kubernetes Local Cluster with k3d, RabbitMQ, KEDA, and Dashboard</h1></strong>

<p>This tutorial guides you through setting up a local Kubernetes cluster using <strong>k3d</strong>, installing <strong>RabbitMQ</strong> for message queuing, configuring <strong>KEDA</strong> for autoscaling, and deploying the <strong>Kubernetes Dashboard</strong> for managing the cluster visually.</p>

<h2>Prerequisites</h2>

<p>Before starting, make sure you have the following tools installed:</p>

<ul>
  <li><a href="https://k3d.io/">k3d</a> - Kubernetes in Docker</li>
  <li><a href="https://kubernetes.io/docs/tasks/tools/">kubectl</a> - Command-line tool for Kubernetes</li>
  <li><a href="https://helm.sh/">Helm</a> - Kubernetes package manager</li>
</ul>

<h3>Installing Helm</h3>

<p>If Helm is not already installed, use the following script:</p>

<pre>
<code>
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
</code>
</pre>

<h2>1. Create a Local Kubernetes Cluster with k3d</h2>

<p>We will create a k3d cluster with 1 server and 2 agents, exposing necessary ports for accessing the Kubernetes Dashboard.</p>

<pre>
<code>
k3d cluster create local-cluster \
  --servers 1 --agents 2 \
  --port "8081:80@loadbalancer" --port "8443:443@loadbalancer"
</code>
</pre>

<p>This will create a local Kubernetes cluster named <code>local-cluster</code>.</p>

<h2>2. Install the Kubernetes Dashboard</h2>

<p>The Kubernetes Dashboard allows you to manage your cluster visually. To install it:</p>

<ol>
  <li>Apply the Kubernetes Dashboard manifest:</li>
</ol>

<pre>
<code>
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
</code>
</pre>

<ol start="2">
  <li>Create an admin user with full cluster privileges for the dashboard by creating a <code>dashboard-adminuser.yaml</code> file with the following content:</li>
</ol>

<pre>
<code>
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
</code>
</pre>

<p>Apply the configuration:</p>

<pre>
<code>
kubectl apply -f dashboard-adminuser.yaml
</code>
</pre>

<ol start="3">
  <li>Start the Kubernetes proxy:</li>
</ol>

<pre>
<code>
kubectl proxy
</code>
</pre>

<ol start="4">
  <li>Access the Dashboard by opening the following URL in your browser:</li>
</ol>

<pre>
<code>
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
</code>
</pre>

<ol start="5">
  <li>Retrieve the login token for the admin user:</li>
</ol>

<pre>
<code>
kubectl -n kubernetes-dashboard create token admin-user
</code>
</pre>

<p>Use this token to log into the dashboard.</p>

<h2>3. Install RabbitMQ with Helm</h2>

<p>RabbitMQ will handle the message queueing for this setup.</p>

<ol>
  <li>Add the Bitnami Helm repository:</li>
</ol>

<pre>
<code>
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
</code>
</pre>

<ol start="2">
  <li>Create a dedicated namespace for RabbitMQ:</li>
</ol>

<pre>
<code>
kubectl create namespace messaging
</code>
</pre>

<ol start="3">
  <li>Install RabbitMQ using Helm:</li>
</ol>

<pre>
<code>
helm install rabbitmq bitnami/rabbitmq --namespace messaging
</code>
</pre>

<ol start="4">
  <li>Verify that RabbitMQ is running:</li>
</ol>

<pre>
<code>
kubectl get pods -n messaging
</code>
</pre>

<h2>4. Install KEDA with Helm</h2>

<p>KEDA (Kubernetes Event-Driven Autoscaling) will automatically scale our pods based on RabbitMQ queue metrics.</p>

<ol>
  <li>Add the KEDA Helm repository:</li>
</ol>

<pre>
<code>
helm repo add kedacore https://kedacore.github.io/charts
helm repo update
</code>
</pre>

<ol start="2">
  <li>Create a namespace for KEDA:</li>
</ol>

<pre>
<code>
kubectl create namespace keda-system
</code>
</pre>

<ol start="3">
  <li>Install KEDA:</li>
</ol>

<pre>
<code>
helm install keda kedacore/keda --namespace keda-system
</code>
</pre>

<ol start="4">
  <li>Verify that KEDA is running:</li>
</ol>

<pre>
<code>
kubectl get pods -n keda-system
</code>
</pre>

<h2>5. Expose RabbitMQ Management Interface (Optional)</h2>

<p>To access RabbitMQ's management UI from outside the cluster, you can expose it using a LoadBalancer:</p>

<ol>
  <li>Create a file called <code>rabbitmq-values.yaml</code> with the following content:</li>
</ol>

<pre>
<code>
service:
  type: LoadBalancer
  port: 5672
  managementPort: 15672
</code>
</pre>

<ol start="2">
  <li>Apply this configuration with Helm:</li>
</ol>

<pre>
<code>
helm upgrade rabbitmq bitnami/rabbitmq --namespace messaging --values rabbitmq-values.yaml
</code>
</pre>

<ol start="3">
  <li>Get the RabbitMQ service IP and port:</li>
</ol>

<pre>
<code>
kubectl get svc -n messaging
</code>
</pre>

<p>Use this information to access the RabbitMQ management UI.</p>

<h2>6. Verifying the Setup</h2>

<p>To ensure everything is working correctly:</p>

<ol>
  <li><strong>Access the Kubernetes Dashboard</strong>:</li>
</ol>

<pre>
<code>
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
</code>
</pre>

<ol start="2">
  <li><strong>Check RabbitMQ</strong>: If exposed, access the RabbitMQ management interface using the external IP and port retrieved earlier.</li>
</ol>

<ol start="3">
  <li><strong>Check the Logs</strong>: View the logs for RabbitMQ and KEDA to ensure they're functioning correctly.</li>
</ol>

<ul>
  <li>RabbitMQ logs:</li>
</ul>

<pre>
<code>
kubectl logs -n messaging -l app=rabbitmq
</code>
</pre>

<ul>
  <li>KEDA logs:</li>
</ul>

<pre>
<code>
kubectl logs -n keda-system -l app=keda
</code>
</pre>

<ol start="4">
  <li><strong>Test Scaling</strong>: Send messages to RabbitMQ and verify that KEDA scales your pods based on the queue length.</li>
</ol>

<hr>

<p>By following this guide, you should have a fully functional local Kubernetes cluster with k3d, RabbitMQ, KEDA for autoscaling, and the Kubernetes Dashboard for easy cluster management.</p>

<p>Feel free to contribute or raise any issues in case of questions or improvements!</p>



