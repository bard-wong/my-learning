# 1.Helm

Helm和charts的主要作用：

- 应用程序封装
- 版本管理
- 依赖检查
- 便于应用程序分发

helm是一个C/S框架的软件，helm相当于一个客户端，tiller是一个 服务端

- Helm CLI 是 Helm 客户端，可以在本地执行
- Tiller 是服务器端组件，在 Kubernetes 群集上运行，并管理 Kubernetes 应用程序的生命周期
- Repository 是 Chart 仓库，Helm客户端通过HTTP协议来访问仓库中Chart的索引文件和压缩包



# 2.Consul

```yaml
global:
  name: consul
  datacenter: dc1
server:
  replicas: 1
  securityContext:
    runAsNonRoot: false
    runAsGroup: 0
    runAsUser: 0
    fsGroup: 0
ui:
  enabled: true
connectInject:
  enabled: true
controller:
  enabled: true
```

# 3.vault

## install

```yaml
server:
  affinity: ""
  ha:
    enabled: true
```

helm install

初始化vault

```shell
kubectl exec vault-0 -- vault operator init -key-shares=1 -key-threshold=1 -format=json > cluster-keys.json
```

生成cluster-keys.json

```json
{
  "unseal_keys_b64": [
    "VE6RoqW6j6xRtazG1OJRU+8ca6Jf2GxUYyRNfNv+Q0c="
  ],
  "unseal_keys_hex": [
    "544e91a2a5ba8fac51b5acc6d4e25153ef1c6ba25fd86c5463244d7cdbfe4347"
  ],
  "unseal_shares": 1,
  "unseal_threshold": 1,
  "recovery_keys_b64": [],
  "recovery_keys_hex": [],
  "recovery_keys_shares": 5,
  "recovery_keys_threshold": 3,
  "root_token": "s.kHkb3QkHEFmeBy6z3FShGU7k"
}
```

> **Insecure operation:** Do not run an unsealed Vault in production with a single key share and a single key threshold. This approach is only used here to simplify the unsealing process for this demonstration.

将unseal_key设为环境变量

```shell
VAULT_UNSEAL_KEY=VE6RoqW6j6xRtazG1OJRU+8ca6Jf2GxUYyRNfNv+Q0c=
```

![image-20210816165820900](C:\Users\a1364\AppData\Roaming\Typora\typora-user-images\image-20210816165820900.png)

此时vault处于Sealing状态需要执行Unseal

```shell
[root@master k8s-yaml]# kubectl exec vault-0 -- vault operator unseal $VAULT_UNSEAL_KEY
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.8.0
Storage Type    consul
Cluster Name    vault-cluster-46023ddb
Cluster ID      e291662b-7b4f-8157-f50a-ebdc565af25c
HA Enabled      true
HA Cluster      https://vault-0.vault-internal:8201
HA Mode         active
Active Since    2021-08-16T08:59:45.993256579Z

```

> **Insecure operation:** Providing the unseal key with the command writes the key to your shell's history. This approach is only used here to simplify the unsealing process for this demonstration.

所有Pod unsealed

![image-20210816170200286](C:\Users\a1364\AppData\Roaming\Typora\typora-user-images\image-20210816170200286.png)

登录到vault

```shell
[root@master k8s-yaml]# cat cluster-keys.json | jq -r ".root_token"
s.kHkb3QkHEFmeBy6z3FShGU7k
[root@master k8s-yaml]# kubectl exec --stdin=true --tty=true vault-0 -- /bin/sh
/ $ vault login
Token (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.kHkb3QkHEFmeBy6z3FShGU7k
token_accessor       NefXw0VyESa6qK432dLY8k8S
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

启用kv-v2 secrets engine

```shell
/ $ vault secrets enable -path=secret kv-v2
Success! Enabled the kv-v2 secrets engine at: secret/
#创建secret
/ $ vault kv put secret/webapp/config username="admin" password="admin"
Key              Value
---              -----
created_time     2021-08-16T09:10:24.854059105Z
deletion_time    n/a
destroyed        false
version          1
```

## First Secret

### Writing a Secret

```shell
/ $ vault kv put secret/hello foo=world
Key              Value
---              -----
created_time     2021-08-16T09:24:51.15599523Z
deletion_time    n/a
destroyed        false
version          1
```

### Getting a Secret

```shell
/ $ vault kv get secret/hello
====== Metadata ======
Key              Value
---              -----
created_time     2021-08-16T09:24:51.15599523Z
deletion_time    n/a
destroyed        false
version          1

=== Data ===
Key    Value
---    -----
foo    world
```

查看指定字段的值

```shell
/ $ vault kv get -field=foo secret/hello
world
```

### Deleting a Secret

```shell
/ $ vault kv delete secret/hello
Success! Data deleted (if it existed) at: secret/hello
```

### Configure Kubernetes authentication

启用一个kubernetes验证

```
vault auth enable kubernetes
```

将kubernetes验证方式设置为使用serviceaccount token，kubernetes_host，ca.crt

```shell
vault write auth/kubernetes/config \
        token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
        kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
		kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

webapp的policy

```shell
vault policy write webapp - <<EOF
path "secret/data/webapp/config" {
  capabilities = ["read"]
}
EOF
Success! Uploaded policy: webapp

```

```shell
vault write auth/kubernetes/role/webapp \
        bound_service_account_names=vault \
        bound_service_account_namespaces=default \
        policies=webapp \
        ttl=24h
Success! Data written to: auth/kubernetes/role/webapp

```

### Launch a web application

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  labels:
    app: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      serviceAccountName: vault
      containers:
        - name: app
          image: burtlo/exampleapp-ruby:k8s
          imagePullPolicy: Always
          env:
            - name: VAULT_ADDR
              value: 'http://vault:8200'
            - name: JWT_PATH
              value: '/var/run/secrets/kubernetes.io/serviceaccount/token'
            - name: SERVICE_PORT
              value: '8080'
```

```
$ curl http://localhost:8080
{"password"=>"admin", "username"=>"admin"}%
```



## Secrets Engines

![Vault Triangle](https://learn.hashicorp.com/img/vault/vault-triangle.png)

### 启用Secrets Engine

```shell
$ vault secrets enable -path=kv kv
Success! Enabled the kv secrets engine at: kv/
```

启用Secrets Engine时path默认为Engine名，所以等同于

```shell
vault secrets enable kv
```

### 查看Secrets Engine

```shell
/ $ vault secrets list
Path          Type         Accessor              Description
----          ----         --------              -----------
cubbyhole/    cubbyhole    cubbyhole_af6d4559    per-token private secret storage
identity/     identity     identity_f3e10ec0     identity store
kv/           kv           kv_8bffa67d           n/a
secret/       kv           kv_70c642e1           n/a
sys/          system       system_0214e6c9       system endpoints used for control, policy and debugging
```

### 禁用 Secrets Engine

```shell
$ vault secrets disable kv/
Success! Disabled the secrets engine (if it existed) at: kv/
```

## Dynamic Secrets

使用Cloud，在使用secrets时访问cloud生成，使用完后立即撤销

#### Enable the AWS secrets engine

```shell
 vault secrets enable -path = aws aws
```

# issue

```
/app/lib/service.rb:43:in `block in <class:ExampleApp>'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1635:in `call'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1635:in `block in compile!'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:987:in `block (3 levels) in route!'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1006:in `route_eval'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:987:in `block (2 levels) in route!'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1035:in `block in process_route'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1033:in `catch'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1033:in `process_route'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:985:in `block in route!'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:984:in `each'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:984:in `route!'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1097:in `block in dispatch!'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1071:in `block in invoke'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1071:in `catch'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1071:in `invoke'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1094:in `dispatch!'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:919:in `block in call!'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1071:in `block in invoke'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1071:in `catch'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1071:in `invoke'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:919:in `call!'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:908:in `call'
	/usr/local/bundle/gems/rack-protection-2.0.7/lib/rack/protection/xss_header.rb:18:in `call'
	/usr/local/bundle/gems/rack-protection-2.0.7/lib/rack/protection/path_traversal.rb:16:in `call'
	/usr/local/bundle/gems/rack-protection-2.0.7/lib/rack/protection/json_csrf.rb:26:in `call'
	/usr/local/bundle/gems/rack-protection-2.0.7/lib/rack/protection/base.rb:50:in `call'
	/usr/local/bundle/gems/rack-protection-2.0.7/lib/rack/protection/base.rb:50:in `call'
	/usr/local/bundle/gems/rack-protection-2.0.7/lib/rack/protection/frame_options.rb:31:in `call'
	/usr/local/bundle/gems/rack-2.1.4/lib/rack/null_logger.rb:11:in `call'
	/usr/local/bundle/gems/rack-2.1.4/lib/rack/head.rb:14:in `call'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:194:in `call'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1950:in `call'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1502:in `block in call'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1729:in `synchronize'
	/usr/local/bundle/gems/sinatra-2.0.7/lib/sinatra/base.rb:1502:in `call'
	/usr/local/bundle/gems/rack-2.1.4/lib/rack/handler/webrick.rb:88:in `service'
	/usr/local/lib/ruby/2.6.0/webrick/httpserver.rb:140:in `service'
	/usr/local/lib/ruby/2.6.0/webrick/httpserver.rb:96:in `run'
	/usr/local/lib/ruby/2.6.0/webrick/server.rb:307:in `block in start_thread'
```

