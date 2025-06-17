# E-Commerce Kubernetes Deployment

This project contains Kubernetes manifests and instructions to deploy a simulated e-commerce platform on AWS EKS.

## Architecture

- **Frontend**: React + NGINX
- **Backend**: Node.js services
- **Database**: MongoDB (StatefulSet)
- **Queue**: RabbitMQ (via Helm)
- **Ingress**: NGINX Controller
- **Observability**: Prometheus, Grafana, EFK Stack
- **Autoscaling**: HPA for frontend

## Deployment Steps


1. Building and tagging the docker images

   docker build -t frontendecom:v1 .

   docker build -t backend:v1 .

   docker login

   docker tag frontendecom:v1 irshad04/frontendecom:v1

   docker tag backend:v1 irshad04/backend:v1

   docker push irshad04/frontendecom:v1
   
   docker push irshad04/backend:v1

2. Create the namespace:
   
   alias k='kubectl'
   k create ns ecommerce
   


3. Deploy backend and frontend:
   k apply -f frontend-deploy.yaml

   Check the deployment
   
   k get all -n ecommerce

   k apply -f backend-deploy.yaml
   k apply -f config.yaml
   k apply -f secret.yaml
   
   Check the deployment
   
   k get all -n ecommerce
   

4. Deploy MongoDB:
   
   k apply -f mongodb.yaml

   Check the deployment
   
   k get all -n ecommerce

5. Deploy RabbitMq:
  
   k apply -f rabbitmq.yaml

   Check the deployment
   
   k get all -n ecommerce
  


6. Setup Ingress:
   
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/aws/deploy.yaml

   In /etc/hosts file I have local host ecommere.local with external ip of the ingress nginx controller.

   k apply -f ingress.yaml

   Accessing the frontend service from the external Ip of the ingress nginx controller.
  
   k get all -n ingress-nginx

   k get svc -n ingress-nginx

  curl -H "Host: ecommerce.local" http://a9a639a3ce4ef416f8aa917542929942-db48f10e3bae29d8.elb.us-east-1.amazonaws.com


7. Enable secutiry policies:
   
   Installing CRDs .
   kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.15/deploy/gatekeeper.yaml

   k apply -f opa-constraint.yaml
   k apply -f opa-template.yaml

8. Enable HPA:
   
   Enabling HPA.

   k apply -f frontend-hpa.yaml

   k get hpa -n ecommerce


   

9. Setup monitoring and logging via Helm.

   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

   helm repo update

   helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack   --namespace monitoring --create-namespace

   k apply -f smon.yaml

   kubectl port-forward svc/kube-prometheus-stack-grafana -n monitoring 3000:80

   helm repo add elastic https://helm.elastic.co
   helm repo update

   helm install elasticsearch elastic/elasticsearch -n logging --create-namespace
   helm install kibana elastic/kibana -n logging
   helm install fluent-bit fluent/fluent-bit -n logging

   kubectl port-forward svc/kibana-kibana -n logging 5601:5601

