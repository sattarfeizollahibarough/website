---
title: سلام Minikube
content_type: tutorial
weight: 5
menu:
  main:
    title: "شروع به کار"
    weight: 10
    post: >
      <p>آماده هستید که دست به کار شویم؟ میخواهیم یک کلاستر ساده کوبرنتیز بسازیم که یک اپلیکیشن را اجرا می‌کند.</p>
card:
  name: tutorials
  weight: 10
---

<!-- overview -->

این آموزش به شما خواهد آموخت که چگونه یک برنامه ساده را با استفاده
کوبرنتیز [Minikube](/docs/setup/learning-environment/minikube) و کاتاکودا اجرا کنید.
کاتاکودا برای شما یک محیط کوبرنتیز مبتنی بر مرورگر وب فراهم می‌اورد.

{{< note >}}
شما همچنین می‌توانید از این مستند نیز بهره بگیرد اگر [Minikube](/docs/tasks/tools/install-minikube/) را بر روی سیستم خود نصب نموده‌اید.
{{< /note >}}



## {{% heading "objectives" %}}


* استقرار یک اپلیکیشن ساده بر روی Minikube.
* اجرای اپلیکیشن.
* بررسی لاگ‌های تولید شده توسط اپلیکیشن.



## {{% heading "prerequisites" %}}


در این آموزش از یک تصویر کانتینری که از NGINX بهره برده است که و به درخواست‌ها پاسخ می‌دهد استفاده شده است.



<!-- lessoncontent -->

## ساخت یک کلاستر Minikube

1. بر روی **Launch Terminal** کلیک کنید

    {{< kat-button >}}

{{< note >}}
    اگر Minikube را بر روی سیستم خود نصب نموده دستور  `minikube start`. را اجرا کنید
{{< /note >}}

2. داشبورد کوبرنتیز را بر روی مرورگر وب خود باز کنید:

    ```shell
    minikube dashboard
    ```

3. فقط در محیط کاتاکودا: در بالای ترمینال بر روی علامت به‌علاوه کلیک کرده و سپس بر روی گزینه **Select port to view on Host 1**. کلیک کنید

4. فقط در محیط کاتاکودا: مقدار `30000` را تایپ کرده و بر روی, **Display Port**. کلیک کنید

## ساخت استقرار

در کوبرنتیز [*Pod*](/docs/concepts/workloads/pods/) به مجموعه‌ای از یک تا چند کانتینر اشاره دارد که با هدف مدیریت واحد آنها بوجود آمده است.
در این آموزش Pod ما فقط شامل یک کانتینر است.
در کوبرنتیز
[*Deployment*](/docs/concepts/workloads/controllers/deployment/) سلامت Pod ها را مورد بررسی قرار می‌دهد و اگر کانتیز یک Pod دچار مشکل شود آن را ریستارت می‌کند.استفاده از Deployment برای ساخت و مقیاس‌دهی Podها توصیه می شوند.

1. از دستور `kubectl create` برای ساخت یک Deployment که یک Pod را مدیریت میکند استفاده می‌شود. سپس Pod کانتینری را بر اساس تصویر کانتینر فراهم شده ایجاد می‌کند

    ```shell
    kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4
    ```

2. بررسی Deployment:

    ```shell
    kubectl get deployments
    ```

    خروجی مشابه ذیل خواهد بود:

    ```
    NAME         READY   UP-TO-DATE   AVAILABLE   AGE
    hello-node   1/1     1            1           1m
    ```

3. بررسی Pod:

    ```shell
    kubectl get pods
    ```

    خروجی مشابه ذیل خواهد بود:

    ```
    NAME                          READY     STATUS    RESTARTS   AGE
    hello-node-5f76cf6ccf-br9b5   1/1       Running   0          1m
    ```

4. بررسی کلاسترها:

    ```shell
    kubectl get events
    ```

5. بررسی تنظیمات `kubectl` :

    ```shell
    kubectl config view
    ```

{{< note >}}
برای اطلاعات بیشتر راجع به دستور `kubectl` این لینک را مشاهده فرمایید [kubectl overview](/docs/reference/kubectl/overview/).
{{< /note >}}

## Create a Service

By default, the Pod is only accessible by its internal IP address within the
Kubernetes cluster. To make the `hello-node` Container accessible from outside the
Kubernetes virtual network, you have to expose the Pod as a
Kubernetes [*Service*](/docs/concepts/services-networking/service/).

1. Expose the Pod to the public internet using the `kubectl expose` command:

    ```shell
    kubectl expose deployment hello-node --type=LoadBalancer --port=8080
    ```

    The `--type=LoadBalancer` flag indicates that you want to expose your Service
    outside of the cluster.

2. View the Service you just created:

    ```shell
    kubectl get services
    ```

    The output is similar to:

    ```
    NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    hello-node   LoadBalancer   10.108.144.78   <pending>     8080:30369/TCP   21s
    kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          23m
    ```

    On cloud providers that support load balancers,
    an external IP address would be provisioned to access the Service. On Minikube,
    the `LoadBalancer` type makes the Service accessible through the `minikube service`
    command.

3. Run the following command:

    ```shell
    minikube service hello-node
    ```

4. Katacoda environment only: Click the plus sign, and then click **Select port to view on Host 1**.

5. Katacoda environment only: Note the 5 digit port number displayed opposite to `8080` in services output. This port number is randomly generated and it can be different for you. Type your number in the port number text box, then click Display Port. Using the example from earlier, you would type `30369`.

    This opens up a browser window that serves your app and shows the app's response.

## Enable addons

Minikube has a set of built-in {{< glossary_tooltip text="addons" term_id="addons" >}} that can be enabled, disabled and opened in the local Kubernetes environment.

1. List the currently supported addons:

    ```shell
    minikube addons list
    ```

    The output is similar to:

    ```
    addon-manager: enabled
    dashboard: enabled
    default-storageclass: enabled
    efk: disabled
    freshpod: disabled
    gvisor: disabled
    helm-tiller: disabled
    ingress: disabled
    ingress-dns: disabled
    logviewer: disabled
    metrics-server: disabled
    nvidia-driver-installer: disabled
    nvidia-gpu-device-plugin: disabled
    registry: disabled
    registry-creds: disabled
    storage-provisioner: enabled
    storage-provisioner-gluster: disabled
    ```

2. Enable an addon, for example, `metrics-server`:

    ```shell
    minikube addons enable metrics-server
    ```

    The output is similar to:

    ```
    metrics-server was successfully enabled
    ```

3. View the Pod and Service you just created:

    ```shell
    kubectl get pod,svc -n kube-system
    ```

    The output is similar to:

    ```
    NAME                                        READY     STATUS    RESTARTS   AGE
    pod/coredns-5644d7b6d9-mh9ll                1/1       Running   0          34m
    pod/coredns-5644d7b6d9-pqd2t                1/1       Running   0          34m
    pod/metrics-server-67fb648c5                1/1       Running   0          26s
    pod/etcd-minikube                           1/1       Running   0          34m
    pod/influxdb-grafana-b29w8                  2/2       Running   0          26s
    pod/kube-addon-manager-minikube             1/1       Running   0          34m
    pod/kube-apiserver-minikube                 1/1       Running   0          34m
    pod/kube-controller-manager-minikube        1/1       Running   0          34m
    pod/kube-proxy-rnlps                        1/1       Running   0          34m
    pod/kube-scheduler-minikube                 1/1       Running   0          34m
    pod/storage-provisioner                     1/1       Running   0          34m

    NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
    service/metrics-server         ClusterIP   10.96.241.45    <none>        80/TCP              26s
    service/kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP       34m
    service/monitoring-grafana     NodePort    10.99.24.54     <none>        80:30002/TCP        26s
    service/monitoring-influxdb    ClusterIP   10.111.169.94   <none>        8083/TCP,8086/TCP   26s
    ```

4. Disable `metrics-server`:

    ```shell
    minikube addons disable metrics-server
    ```

    The output is similar to:

    ```
    metrics-server was successfully disabled
    ```

## Clean up

Now you can clean up the resources you created in your cluster:

```shell
kubectl delete service hello-node
kubectl delete deployment hello-node
```

Optionally, stop the Minikube virtual machine (VM):

```shell
minikube stop
```

Optionally, delete the Minikube VM:

```shell
minikube delete
```



## {{% heading "whatsnext" %}}


* Learn more about [Deployment objects](/docs/concepts/workloads/controllers/deployment/).
* Learn more about [Deploying applications](/docs/tasks/run-application/run-stateless-application-deployment/).
* Learn more about [Service objects](/docs/concepts/services-networking/service/).
