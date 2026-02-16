All Sentinet components can be hosted in Kubernetes. These scripts have beed tested to successfully run in the local Docker Desktop Kubernetes cluster, as well as in Azure Kubernetes Service.

To run Sentinet in Docker Desktop:

1. Download and install Docker Desktop: https://www.docker.com/products/docker-desktop/
2. Enable Kubernetes in Docker Desktop ("Settings" -> "Kubernetes" -> "Enable Kubernetes [X]")
3. Install the "Docker\config\host.docker.internal.crt" certificate into Windows Trusted Authorities store.
5. Run "run.cmd" script. The script will create the following resources in Kubernetes:
* *"sentinet"* namespace.
* *"database"* deployment hosting a single pod running SQL Server database.
* *"database-svc"* ClusterIP service that exposes "database" pod to internal cluster network (database-svc.sentinet.svc.cluster.local:1433).
* *"agent-config"* config-map containing agent.json Agent configuration file.
* *"agent"* deployment hosting one or more replicas of a pod running Agent. The pod maps "agent-config" as a volume to access configuration file.
* *"repository-config"* config-map containing repository.json Repository configuration file and host.docker.internal.pfx X.509 certificate file.
* *"repository"* deployment hosting one or more replicas of a pod running Repository. The pod maps "repository-config" as a volume to access configuration and other files.
* *"repository-svc"* LoadBalancer service that exposes "repository" pod(s) to external network (http://host.docker.internal:7000 and https://host.docker.internal:7001).
* *"node-config"* config-map containing node.json Node configuration file and host.docker.internal.pfx X.509 certificate file.
* *"node"* deployment hosting one or more replicas of a pod running Node. The pod maps "node-config" as a volume to access configuration and other files.
* *"node-svc"* LoadBalancer service that exposes "node" pod(s) to external network (http://host.docker.internal:5000 and https://host.docker.internal:5001).
6. To check the status of pods, execute: "kubectl get pods -n sentinet". You shall see at least 4 pods in Running state.
7. To stop and remove all resources run "remove.cmd" script.
