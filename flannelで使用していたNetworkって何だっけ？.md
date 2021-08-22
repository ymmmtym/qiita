---
title: flannelで使用していたNetworkって何だっけ？
tags: kubernetes flannel
author: yumenomatayume
slide: false
---
と思ったときに叩くコマンドです。

## その1: Podでコマンドを実行して確認する

flannelで使用するセグメントは、Podにある`/etc/kube-flannel/net-conf.json`に記載されています。

```bash
kubectl exec -it -n kube-system $(kubectl get pods -n kube-system | grep kube-flannel | cut -f 1 -d ' ') -- cat /etc/kube-flannel/net-conf.json
```

出力

```
Defaulted container "kube-flannel" out of: kube-flannel, install-cni (init)
{
  "Network": "10.244.0.0/16",
  "Backend": {
    "Type": "vxlan"
  }
}
```

## その２: ConfigMapを確認

```bash
root@k8s-master1:~# kubectl get cm -n kube-system kube-flannel-cfg -o json | jq '.data["net-conf.json"]'
```

出力(defaultで改行コード入ります)

```
"{\n  \"Network\": \"10.244.0.0/16\",\n  \"Backend\": {\n    \"Type\": \"vxlan\"\n  }\n}\n"
```

## その他

```bash
root@k8s-master1:~# kubectl get cm -n kube-system kube-flannel-cfg -o jsonpath --template {'.data["net-conf.json"]'}
error: error parsing jsonpath {.data["net-conf.json"]}, invalid array index "net-conf.json"
```

jqコマンドと同じようにはいきませんでした。


[[ jq ] keyの値にドット(.)が入っているときはどうするか - Qiita](https://qiita.com/penguin_dream/items/18a414302ccc301f8b54)

