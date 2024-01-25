**Deploy three different Apps on k8s cluster**

kubectl apply -f https://raw.githubusercontent.com/puneetgavri/kubernetes-world/main/installation/manifest_files/ingress/deployApps.yml

**Deploy Ingress Controller**

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.0/deploy/static/provider/baremetal/deploy.yaml

**Deploy Ingress Resource(rules)**

kubectl apply -f https://raw.githubusercontent.com/puneetgavri/kubernetes-world/main/installation/manifest_files/ingress/ingressRule.yml

**Get Ingress Controller Service NodePort**

  kubectl get ingress
  
  kubectl describe ingress <ingress-name>
  
  kubectl get services -n ingress-nginx

  
**validate
   From the above details noted in your browser hit as below**
   
   http://any node public ip:NodePort/app1

   
