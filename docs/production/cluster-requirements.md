# Kubernetes/OpenShift 集群

本部分提供有关 Rasa Pro 架构的信息，以及将 Rasa Pro 对话机器人部署到 Kubernetes 或 OpenShift 集群的说明。

!!! info "注意"

    如果你不熟悉将微服务应用程序部署到 Kubernetes 或 OpenShift 集群，我们强烈推荐 [Rasa 的托管服务](https://rasa.com/product/managed-service/)。

## 集群要求 {#cluster-requirements}

你需要一个现有的 [Kubernetes 集群](https://kubernetes.io/)或 [OpenShift 集群](https://www.openshift.com/)。如果还没有，可以从云提供者处获取托管集群，例如：

- [Amazon EKS](https://aws.amazon.com/eks/)
- [Microsoft Azure](https://azure.microsoft.com/en-us/services/kubernetes-service/)
- [Google Cloud](https://cloud.google.com/kubernetes-engine)
- [DigitalOcean](https://www.digitalocean.com/products/kubernetes/)
