# Rasa Pro Helm Chart

Rasa Pro Helm Chart 托管在我们的 GCP Artifact Registry 上。本页介绍如何下载 Chart。

Rasa Pro Helm Chart 托管在 GCP 上公开可用的工件注册表中。要下载 Rasa Pro Helm Chart，请运行以下命令：

```shell
helm pull oci://europe-west3-docker.pkg.dev/rasa-releases/helm-charts/rasa
```

这将下载一个包含 Helm Chart 的 `rasa-x.y.z.tgz` 文件。
