# AWS Loadblancer controller in China Region

## prerequsite

1. Kubernets version >= 1.17
2. Enough right to provision required resource(**To be continued**)

## Step by Step Introduction

#### Provision ServiceAccount and iam for Loadblancer controller

* Download repo

```
git clone https://github.com/jansony1/cn-container-lab.git
cd cn-container-lab/lb-controller
```

* Enable oidc on eks, replace **\<Cluster-Name>** with **your cluster name**

```
eksctl utils associate-iam-oidc-provider \
    --region cn-northwest-1 \
    --cluster <Cluster-Name> \
    --approve
```

* Configure  iam role for controller to use

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy-cn.json
```

* Create serviceAccount and its mapping to iam role，replace **\<Cluster name>** and **<account_id>**

```
eksctl create iamserviceaccount \
	--cluster=<your cluster name> \
	--namespace=kube-system \
	--name=aws-load-balancer-controller \
	--attach-policy-arn=arn:aws-cn:iam::<account_id>:policy/AWSLoadBalancerControllerIAMPolicy \
	--approve
```

#### Install certManager

* Modify image repo to [NWCD mirror](https://github.com/nwcdlabs/container-mirror)

```
sed -i "s#quay.io#048912060910.dkr.ecr.cn-northwest-1.amazonaws.com.cn/gcr#g" cert-manager.yaml
```

* Install 

```
kubectl apply -f cert-manager.yaml
```

* Wait unit Cert manager status  to ready

```
[ec2-user@ip-10-0-1-58 alb_v2]$ kubectl get pods -ncert-manager
NAME                                      READY   STATUS    RESTARTS   AGE
cert-manager-74fdd7bc8f-6hrzp             1/1     Running   0          20h
cert-manager-cainjector-8665f9998-zzsfz   1/1     Running   0          20h
cert-manager-webhook-677b8cbf67-6pr5z     1/1     Running   0          20h
```

#### Install  Loadblancer controller

* Modiy yaml to match your cluster, **Replace \<Your-cluster-name> to your cluster name** 

```
sed -i "s/<Cluster-Name>/<Your-cluster-name>/g" lb-controller-v2.yaml
```

* Deploy lb-controller

```
kubectl apply -f lb-controller-v2.yaml
```

#### Install  2048 example

* Install [2048 Game](https://docs.amazonaws.cn/en_us/eks/latest/userguide/alb-ingress.html)

```
kubectl apply -f 2048_full.yaml
```

* Verify alb has shown up 

```
[ec2-user@ip-10-0-1-58 alb_v2]$ kubectl get ing --all-namespaces
NAMESPACE   NAME           HOSTS   ADDRESS                                                                          PORTS   AGE
game-2048   ingress-2048   *       k8s-game2048-ingress2-eaaf52e61e-216201375.cn-northwest-1.elb.amazonaws.com.cn   80      19h
```









