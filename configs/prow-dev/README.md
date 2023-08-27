# prow-config

Prow dev configs for openGemini community.

> 注意： 生产环境至少准备200G磁盘，一个给ghproxy,一个给minio

# 第一步 创建[Github apps](https://github.com/settings/apps)

参考：https://docs.prow.k8s.io/docs/getting-started-deploy/#github-app


# 第二步 创建[Github auth apps](https://github.com/settings/developers)

参考：https://docs.prow.k8s.io/docs/components/core/deck/github-oauth-setup/

# 第三步 创建namespace

```bash
make prow-namespace
make test-pods-namespace
```

# 第四步 准备一些密钥

1. hmac
```bash
openssl rand -hex 20 > ./hmac-token

cp ./example/hmac-token.example.yaml ./cluster/hmac-token.yaml

替换其中的 << insert-hmac-token-here >> 为 ./hmac-token中的内容
```

2. github-token

App ID: 349173

点击 `Generate a private key`, 并下载private-key.pem到本地

> private-key.pem

```bash
cat ./private-key.pem

cp ./example/github-token.example.yaml ./cluster/github-token.yaml 

替换其中的 << insert-token-here >> 为 ./private-key.pem 中的内容
```

# 第五步 准备一些config文件

## secret文件

1. github-oauth-config

```bash
修改./github-oauth-config.yaml文件中的变量
kubectl -n prow create secret generic github-oauth-config --from-file=secret=./github-oauth-config.yaml
```

2. cookie

```bash
openssl rand -out cookie.txt -base64 32

kubectl -n prow create secret generic cookie --from-file=secret=./cookie.txt
```

3. oauth-token() (创建[personal access token](https://github.com/settings/tokens/new))

选repo权限即可

```bash
echo ghp_rJcBUpAFCzLItP20...... > oauth-token

**注意这里不是 =secret= 而是=oauth=**

kubectl -n prow create secret generic oauth-token --from-file=oauth=./oauth-token
```

## config文件

1. 修改config/config.yaml
2. 修改config/plugins.yaml
3. 修改cluster/starter.yaml

把starter.yaml中 prow.openGemini.org 替换成你自己的本机域名，或是公有云域名

修改starter.yaml中的变量， << MINIO_ADMIN_USERNAME >>, << MINIO_ADMIN_PASSWORD >>，请自定义一个安全的账号名和密码。

4. Optionally, you can update the cert-manager.io/cluster-issuer: annotation if you use cert-manager

# 第六步 安装ProwJob的CRD

> github: test-infra/config/prow/cluster/prowjob_customresourcedefinition.yaml

kubectl apply --server-side=true -f ./jobs/prowJob/prowjob_customresourcedefinition.yaml

# 第七步 配置ghproxy

> github: test-infra/config/prow/cluster/pushgateway_deployment.yaml

修改pushgateway_deployment.yaml中所有的namespace: default为prow

在starter.yaml中的ghproxy的deployment中添加下面的参数

`--legacy-disable-disk-cache-partitions-by-auth-header=false`

kubectl apply -f ./cluster/pushgateway_deployment.yaml

# 第八步 启动starter

```bash
kubectl apply -f ./cluster/starter.yaml
```

# 第九步 安装Gihub app到repo中
安装Github app到repo中，在github页面上操作

在github->组织-> settings页面 -> installed Github Apps -> 操作你建立的github app -> Configure -> 选择仓库

# 第十步 搞定带color的label

# 第十一步 在公有云上部署Prow


# troubleshot

## 首次贡献代码的开发者的PR，为什么没有自动merge？
第一次贡献代码的开发者，必须找committer手动合并代码后，成为了contributor后，tide才能自动合并此开发者新的PR