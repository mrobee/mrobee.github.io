---
title: "Use SparkKubernetesOperator on Airflow 1.10.x"
date: 2021-12-31T21:56:12+07:00
---

For many organizations, upgrading the Airflow installation to the newest version of 2.x is not an easy choice. Fortunately, the Airflow maintainer provides us with the backports. So, we can leverage them.

In our case, we need to use the new [SparkKubernetesOperator](https://airflow.apache.org/docs/apache-airflow-providers-cncf-kubernetes/stable/_api/airflow/providers/cncf/kubernetes/operators/spark_kubernetes/index.html) which is available in the Airflow version 2.x. This operator provides us with the ability to launch a SparkApplication resource using the [Spark On Kubernetes Operator](https://github.com/GoogleCloudPlatform/spark-on-k8s-operator). Our Airflow version was at 1.10.12 and we want to upgrade it to 1.10.15, the upgrade to this latest version went smooth without any problem. In theory, any Airflow 1.10.x can install the backport packages.

There's a problem when we tried to use this operator. It needs a `kubernetes_conn_id`.

### Create Kubernetes Connection
To create the kubernetes connection in the Airflow, we can't use the Airflow UI, since the `kubernetes` connection type doesn't come up in the list.

I tried to use the environment variable `AIRFLOW_CONN_KUBERNETES_DEFAULT`, it doesn't work either.

The only way to create the connection is through the Airflow CLI. Since our Airflow cluster is already in the same cluster with the [Spark On Kubernetes Operator](https://github.com/GoogleCloudPlatform/spark-on-k8s-operator), I use the `in_cluster` option to create the connection. You can refer to the [documentation](https://airflow.apache.org/docs/apache-airflow-providers-cncf-kubernetes/stable/connections/kubernetes.html) to choose which option is suitable for you.

Here's the Airflow CLI command to create the connection.

```bash
airflow connections add --conn-extra='{"kubernetes__in__cluster":true,"kubernetes__namespace":"the-default-namespace-to-use"}' --conn-host='' --conn-type=kubernetes kubernetes_default
```

Basically, we need to provide the `--conn-type=kubernetes`, `--conn-host=''`, the connection ID, and the connection extras. If you decide to use other options, you need to provide that option in the extras.

To use the kube config path, provide the extras with

```json
{ "kubernetes__kube_config_path": "path_to_kube_config" }
```

Or if you want, you can pass the content of the kube config directly in the extras

```json
{ "kubernetes__kube_config": "content_of_kube_config_in_json_format" }
```

That's it! Now we're ready to use the SparkKubernetesOperator.
