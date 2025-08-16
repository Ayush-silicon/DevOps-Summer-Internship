```shell

kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://hpa-demo-deployment; done"

```


CLusterAutoscalerPOlicy. 

```json

{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Action": [
              "autoscaling:DescribeAutoScalingGroups",
              "autoscaling:DescribeAutoScalingInstances",
              "autoscaling:DescribeLaunchConfigurations",
              "autoscaling:DescribeTags",
              "autoscaling:SetDesiredCapacity",
              "autoscaling:TerminateInstanceInAutoScalingGroup",
              "ec2:DescribeLaunchTemplateVersions"
          ],
          "Resource": "*",
          "Effect": "Allow"
      }
  ]
}

```

# Commands to setup prometheus in the cluster. for this install ebs-csi-driver, give ec2fullaccess.

```shell
kubectl get --raw /metrics
kubectl create namespace prometheus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm upgrade -i prometheus prometheus-community/prometheus --namespace prometheus --set alertmanager.persistentVolume.storageClass="gp2",server.persistentVolume.storageClass="gp2"

```

Check if installation is successful:

```shell
kubectl get pods -n prometheus
```
You can port-forward this using below command to your localhost

```shell
kubectl --namespace=prometheus port-forward deploy/prometheus-server 9090
```
```shell
kubectl create namespace grafana
helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana grafana/grafana --namespace grafana --set persistence.storageClassName="gp2" --set persistence.enabled=true --set adminPassword='admin123' --values
```

You can add a dashboard for the node by - Dashboard -> Import -> 1860 -> Load

# other Autoscaler Blog link:- 

https://medium.com/@sibasishsatpathy2002/mastering-kubernetes-autoscaling-hpa-vpa-and-keda-explained-ea0975d9012f


