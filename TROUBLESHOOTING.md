# Troubleshooting Guide

## Issue: Frontend Not Accessible

- Verify ingress rules: `kubectl describe ingress ecommerce-ingress`

  k describe ingress ecommerce-ingress -n ecommerce

- Confirm DNS/IP mapping for `ecommerce.local`.

  curl -H "Host: ecommerce.local" http://a9a639a3ce4ef416f8aa917542929942-db48f10e3bae29d8.elb.us-east-1.amazonaws.com

  

## Issue: Backend-Mongo Intermittent

- Use `kubectl logs` and `exec` to diagnose connectivity.
  
  k get all -n ecommerce

  k exec -it frontend-55684c8c66-4gxl6 -n ecommerce -- sh

  curl http://192.168.52.114:80

- Ensure MongoDB pod is Ready and StatefulSet is stable.

  ping mongodb

  nc -zv rabbitmq 5672

  curl -u guest:guest http://rabbitmq:15672/api/overview
