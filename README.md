## Elasticsearch K8S Single Cluster

### Pre-requesites
+ Docker is installed
+ Kubernetes is installed with:
  + kops
  + kubectl
+ Kubectl config in $HOME user directory

### Topology
  + Single Cluster: elk.blockaidservices.com
    + 3 K8S Master nodes
    + 10 K8S Worker nodes for:
      + 3 ELK Master Nodes (1 pod per Node)
      + 3 ELK Data Nodes (1 pod per Node)
      + 2 ELK Coordinating Nodes (1 pod per Node for READ LB)
      + 1 ELK Kibana instance
      + 1 ELK Logstash instance
```
******* Start ELASTICSEARCH ******

1. Go to generated templates
> cd /BLOCK-AID/github/aws-k8s-elk

2. Apply the namespace file and create AWS storage (EBS).
> kubectl apply -f 1_elastic/init
> kubectl get namespaces
> kubectl describe sc --namespace=k8s-elastic

3. Create 3 ELK Master Nodes, 3 ELK Data Nodes, and 2 Coordinating Nodes.  Verify its creation.
> kubectl apply -f 1_elastic
> kubectl get all --namespace=k8s-elastic
> kubectl rollout status sts/elasticsearch-data --namespace=k8s-elastic
> kubectl describe pod elasticsearch-coord-xxxxxxxxxxx --namespace=k8s-elastic
> kubectl describe pod/elasticsearch-data-0 --namespace=k8s-elastic
> kubectl get pods -o wide    //Check if assignments pods-nodes were successful
> kubectl --namespace=k8s-elastic describe service elasticsearch-coord

4. Check that your Elasticsearch cluster is functioning correctly by performing a request against the REST API:
> kubectl port-forward elasticsearch-data-0 9200:9200 --namespace=k8s-elastic

In a separate terminal, run:
> curl http://localhost:9200/_cluster/state?pretty
> curl http://localhost:9200/_cluster/health?pretty
> curl http://localhost:9200/_cat/nodes?v


******* Start KIBANA ******

1. create a Service and Deployment
> kubectl apply -f 2_kibana

2. Check and see if all your Kibana pod ready:
> kubectl get pods --namespace=k8s-elastic
> kubectl get all --namespace=k8s-elastic

 Then forward the local port 5601 to port 5601 on this Pod:
> kubectl port-forward kibana-8495d4f8dd-wt4m7 5601:5601 --namespace=k8s-elastic

Then try browser: http://localhost:5601


******* Start LOGSTASH ******

1. create a Service and Deployment
> kubectl apply -f 3_logstash
> kubectl get all --namespace=k8s-elastic

*** to undo> kubectl delete -f 3_logstash

2. find the public IP for LOGSTASH
> kubectl describe service/logstash --namespace=k8s-elastic | grep Ingress

LoadBalancer Ingress:     a0c8ab97e2a6a11e9a1aa0683a13067e-130054029.us-east-2.elb.amazonaws.com

Now that we have an IP address, we can set up DNS for the address we choose earlier during the setup

3. Once it is setup correctly, a query for the host address should return the LoadBalancer Ingress’s IP address:
> dig logstash.blockaidservices.com

Check browser: https://kibana.rccl-dns.com
```
