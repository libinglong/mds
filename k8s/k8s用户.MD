#### k8s 用户

> https://kubernetes.io/docs/reference/access-authn-authz/authentication/
>
> All Kubernetes clusters have two categories of users: service accounts managed by Kubernetes, and normal users.

根据官方文档，可以将用户分为两类

* 由k8s集群管理的service account

  service account可以通过k8s api创建

* 普通用户

  k8s中没有关于普通用户的对象，因此无法通过k8s api创建。任何提供由k8s的CA证书签发的证书的客户端，对于k8s来说都是属于该类用户。证书中的subject字段，描述了用户名称和用户所属的组。

  > For example, using the `openssl` command line tool to generate a certificate signing request:
  >
  > ```bash
  > openssl req -new -key jbeda.pem -out jbeda-csr.pem -subj "/CN=jbeda/O=app1/O=app2"
  > ```
  >
  > 比如上述命令生成的证书中的subject字段中，用户名就是jbeda，用户属于app1和app2
  >
  > 参考下文如何获取有效的证书

#### 创建用户

* 创建service account

  直接通过k8s api创建，该类用户不能指定group字段，但是有默认的分组

  > https://kubernetes.io/docs/reference/access-authn-authz/authentication/
  >
  > Service accounts authenticate with the username `system:serviceaccount:(NAMESPACE):(SERVICEACCOUNT)`, and are assigned to the groups `system:serviceaccounts` and `system:serviceaccounts:(NAMESPACE)`

* 创建普通用户

  * 如何签发客户端证书

    必须使用k8sCA证书签发的证书才能通过认证，使用ca证书签发的子证书再次进行签发，是不会通过认证的。

    参考[创建用户](https://www.adaltas.com/en/2019/08/07/users-rbac-kubernetes/)

  * mac中minikube对应位置

    ~/.minikube/ca.crt

    ~/.minikube/ca.key

  * kubeadm安装的k8s集群

    参考官方文档[证书位置](https://kubernetes.io/docs/setup/best-practices/certificates/#where-certificates-are-stored)

#### 如何防止权限扩散

* 对于service account而言，其权限可以通过k8s api的rbac控制，这里不再赘述

* 对于普通用户，是通过CA证书签发来完成创建过程，创建过程需要用到的核心的两个文件就是ca.crt和ca.key，ca.crt是要分发给客户端做tls认证用的，因此必然会分发出去，而ca.key则不需要分发，因此集群管理员必须要注意，不要泄露ca.key.

  > 一般kubeconfig中都可以找到k8s的ca证书，一般是base64格式或者配置一个文件位置









