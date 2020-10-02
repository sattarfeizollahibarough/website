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

## ساخت یک سرویس

به صورت پیش‌فرض یک Pod تنها به وسیله IP داخلی خودش که در کلاستر کوبرنتیز تعریف شده است قابل دسترسی است.
برای اینکه کانتینر `hello-node` را قابل دسترسی از خارج شبکه مجازی کوبرنتیز بکنید لازم است که Pod مذکور را
توسط یک[*Service*](/docs/concepts/services-networking/service/) در اختیار کاربران قرار دهید.

1. دستور `kubectl expose` یک Pod را به صورت عمومی در اینترنت ارائه میکند:

    ```shell
    kubectl expose deployment hello-node --type=LoadBalancer --port=8080
    ```

    گزینه `--type=LoadBalancer` مشخص می‌کند که شما قصد دارید تا سرویس مذکور را خارج از کلاستر در دسترس قرار دهید.

2. برای بررسی سرویسی که در حال حاضر آن را در دسترس قرار داده‌اید گزینه زیر استفاده می‌شود:

    ```shell
    kubectl get services
    ```

     خروجی مشابه ذیل خواهد بود:

    ```
    NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    hello-node   LoadBalancer   10.108.144.78   <pending>     8080:30369/TCP   21s
    kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          23m
    ```

    روی بسترهایی که توسط فراهم‌کننده زیرساخت ابری فراهم شده است،
    یک IP خارجی برای دسترسی به سرویس تدارک دیده می‌شود. در Minikube نوع
    `LoadBalancer` باعث ایجاد دسترسی به سرویس توسط دستور `minikube service`
    خواهد شد.

3. دستور ذیل را اجرا نمایید:

    ```shell
    minikube service hello-node
    ```

4. فقط در محیط کاتاکودا: بر روی گزینه بلاوه کلیک کرده و سپس بر روی **Select port to view on Host 1** کلیک کنید.

5.  یک عدد پنج رقمی برای پورت ارائه شده است. فقط در محیط کاتاکودا: توجه داشته باشید که در مقابل  `8080`. این عدد به صورت رندم تولید می‌شود و ممکن است این عدد برای شما متفاوت باشد. عددی را که به عنوان شماره پورت ایجاد شده است را در باکس تایپ کنید. برای مثال در این محیط باید عدد `30369`. را تایپ کنید.

    با انجام این کار یک صفحه وب برای شما باز می‌شود که پاسخ سرور را برای شما بازگردانده است.

## فعال سازی addons

Minikube دارای چندین {{< glossary_tooltip text="addons" term_id="addons" >}} فعال، غیرفعال کرد. داخلی است که می‌توان آنها را 

1. برای دریافت لیست addons های جاری از دستور ذیل استفاده میشود:

    ```shell
    minikube addons list
    ```

     خروجی مشابه ذیل خواهد بود:

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

2. برای فعال سازی addon `metrics-server` دستور ذیل صادر می‌شود:

    ```shell
    minikube addons enable metrics-server
    ```

     خروجی مشابه ذیل خواهد بود:

    ```
    metrics-server was successfully enabled
    ```

3. مشاهده Pod و سرویسی که در حال حاضر ساخته اید:

    ```shell
    kubectl get pod,svc -n kube-system
    ```

     خروجی مشابه ذیل خواهد بود:

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

4. غیرفعال کردن `metrics-server`:

    ```shell
    minikube addons disable metrics-server
    ```

     خروجی مشابه ذیل خواهد بود:

    ```
    metrics-server was successfully disabled
    ```

## پاک‎سازی

حالا می‌توانید منابعی که اختصاص داده اید را پاکسازی کنید:

```shell
kubectl delete service hello-node
kubectl delete deployment hello-node
```

به صورت اختیاری می‌توانید ماشین مجازی Minikube را متوقف کنید:

```shell
minikube stop
```

به صورت اختیاری می‌توانید ماشین مجازی Minikube را حذف کنید:

```shell
minikube delete
```



## {{% heading "whatsnext" %}}


* مطالعه بیشتر در مورد [Deployment objects](/docs/concepts/workloads/controllers/deployment/).
* مطالعه بیشتر در مورد [Deploying applications](/docs/tasks/run-application/run-stateless-application-deployment/).
* مطالعه بیشتر در مورد [Service objects](/docs/concepts/services-networking/service/).
