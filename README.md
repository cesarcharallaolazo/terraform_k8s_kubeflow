[terraform_apply_finished]:img/terraform_apply_finished.png
[verify_eks_cluster]:img/verify_eks_cluster.png
[kubeflow_installation]:img/kubeflow_installation.png
[dex_login]:img/dex_login.png
[kubeflow_central_dashboard]:img/kubeflow_central_dashboard.png

# Provision a Kubeflow Cluster

In this demo we have installed Kubeflow on AWS EKS.

Soon, there will be instructions for other clouds (GCP, Azure).

## Versions

- Terraform: v1.2.7
- EKS: 1.21
- Kubeflow: v1.5.1
- Kustomize: 3.2.0
- awslabs/kubeflow-manifests: v1.5.1-aws-b1.0.1

## Steps

### A. Provision an EKS cluster

1. Install Terraform
2. Set your AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY (set this at ~/.aws/credentials file or similar path for Linux)
3.      terraform init
4.      terraform plan
5.      terraform apply
![alt][terraform_apply_finished]

6. wait until EKS cluster and dependencies has been created at AWS - 10 minutes roughly
7. then update kubeconfig:

      aws eks --region $(terraform output -raw region) update-kubeconfig \
    --name $(terraform output -raw cluster_name)

8. verify eks cluster

        kubectl cluster-info
![alt][verify_eks_cluster]

### B. Install Kubeflow

9.      cd kubeflow-manifests
10.      sh install_kubeflow_on_k8s.sh
![alt][kubeflow_installation]

11. if fails, retry above step until it works
12. verify all Pods are ready before trying to connect:

        kubectl get pods -n cert-manager
        kubectl get pods -n istio-system
        kubectl get pods -n auth
        kubectl get pods -n knative-eventing
        kubectl get pods -n knative-serving
        kubectl get pods -n kubeflow
        kubectl get pods -n kubeflow-user-example-com
        # Depending on your installation if you installed KServe
        kubectl get pods -n kserve

13. to get started quickly, you can access Kubeflow via port-forward. Run the following to port-forward Istio’s Ingress-Gateway to local port 8080

    kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80

14. now you can access the Kubeflow Central Dashboard at http://localhost:8080 (Dex login screen)
15. login with the default user’s credential: 
    - user: user@example.com
    - password: 12341234

![alt][dex_login]

![alt][kubeflow_central_dashboard]

16. if localhost:8080 is not reached, retry from step 10 until it works 

## Destroy All (Kubeflow + EKS Cluster)

1.      kubectl get profile
        kubectl delete profile --all

2. at 'kubeflow-manifests' directory, run

        sh uninstall_kubeflow_on_k8s.sh

3. at repo root path, run
        
        terraform destroy

## Note

This demo was ran into an MacOS-Intel darwin_amd64

## Resources

- https://learn.hashicorp.com/tutorials/terraform/eks
- https://awslabs.github.io/kubeflow-manifests/docs/deployment/prerequisites/
- https://awslabs.github.io/kubeflow-manifests/docs/deployment/vanilla/guide/
- https://awslabs.github.io/kubeflow-manifests/docs/deployment/uninstall-kubeflow/