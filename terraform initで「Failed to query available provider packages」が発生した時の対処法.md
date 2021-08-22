---
title: terraform initで「Failed to query available provider packages」が発生した時の対処法
tags: Terraform ESXi
author: yumenomatayume
slide: false
---
自宅サーバ用途でESXiを導入しました。WebからポチポチしてVMを立てるのは大変なので、terraformを使ってコードで管理しようと思います。

使用したプラグインは以下になります。

- Registry: [josenk/esxi | Terraform Registry](https://registry.terraform.io/providers/josenk/esxi/latest)
- GitHub: [josenk/terraform-provider-esxi: Terraform-provider-esxi plugin](https://github.com/josenk/terraform-provider-esxi)

使用しているterraformのversionは以下になります。(Mac環境を使用しています)

```bash
$ terraform -v
Terraform v0.15.4
on darwin_amd64

```

## エラー内容

はじめに`terraform init`で使用するprovider pluginの設定を行いますが、この時点でエラーが発生しました。

```bash
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/esxi...
╷
│ Error: Failed to query available provider packages
│ 
│ Could not retrieve the list of available versions for provider hashicorp/esxi: provider registry registry.terraform.io does not have a
│ provider named registry.terraform.io/hashicorp/esxi
│ 
│ All modules should specify their required_providers so that external consumers will get the correct providers when using a module. To see
│ which modules are currently depending on hashicorp/esxi, run the following command:
│     terraform providers

```

使用したファイルは以下になります。

```main.tf
terraform {
  required_providers {
    esxi = {
      source = "hashicorp/esxi"
    }
  }
}

provider "esxi" {
  esxi_hostname      = "192.168.0.2" # ESXiのIPアドレス
  esxi_hostport      = "22"
  esxi_hostssl       = "443"
  esxi_username      = "root" # ESXiにSSHするときのUSER
  esxi_password      = "pa$$w0rd" # ESXiにSSHするときのPASSWORD
}

resource "esxi_guest" "vmtest" {
  guest_name         = "vmtest"
  disk_store         = "datastore1"
  network_interfaces {
    virtual_network = "VM Network"
    mac_address     = "00:50:56:00:00:00"
  }
}
```

## 対処法

`source = "hashicorp/esxi"`という指定が間違っていたので、`source = "josenk/esxi"`に修正して対処しました。(公式ドキュメントにも書いてありました、、)

```main.tf
terraform {
  required_providers {
    esxi = {
      source = "josenk/esxi" # ここを修正
    }
  }
}
```

terraform 0.13からProviderが https://registry.terraform.io/ から配布されるので、`source = "<Provider名>/<プラグイン名>"`と記載することで https://registry.terraform.io/ からプラグインを探しにいきます。

また、`source = "registry.terraform.io/josenk/esxi"`と記載しても同じ動作になります。

プラグインを使用する際には、レジストリのWebページにアクセスして右上の「USE PROVIDER」をクリックすると、必要なterraformのversionとコードが見れるので、表示された通りに記載すれば問題ないです。

![terraform-registry-josenk-esxi.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/251749/f4ef1acb-4bc3-280e-93c8-1ae32a05b796.png)

## Reference

- [terraform 0.13でローカルproviderを利用する方法 - Sionの技術ブログ](https://sioncojp.hateblo.jp/entry/2020/10/13/195117)
- [could not query provider registry for registry.terraform.io でterraformのproviderがダウンロードできない問題の対処 - Screaming Loud](https://yuutookun.hatenablog.com/entry/2021/01/05/192708)



