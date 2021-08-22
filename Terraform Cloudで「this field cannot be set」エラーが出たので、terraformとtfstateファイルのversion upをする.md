---
title: Terraform Cloudで「this field cannot be set」エラーが出たので、terraformとtfstateファイルのversion upをする
tags: Terraform TerraformCloud Cloud IaC
author: yumenomatayume
slide: false
---
GitHubリポジトリとTerraform Coudを連携させて、OCI(Oracle Cloud Infrastructure)のリソースを管理しているものがある。

Terraform Cloudはクラウド上でTerraformが自動的に実行されるサービスであり、
GitHubと連携すると以下の処理が走る。

- PR作成時に`terraform plan`が実行される
- masterブランチにpush or mergeされると`terraform apply`が実行される

この時、いちいちPR作成してTerraform Cloudで実行されるのを待つのが面倒だったため、
Terraform Cloudから`tfstate`ファイルをダウロードして、ローカルPCで`terraform plan/apply`を実行していた。

この時に「this field cannot be set」というエラーが発生した。

## Terraformのversion up

まず、terraformの公式サイト通りにtfファイルを作成していたが、`terraform apply`でエラーとなった。

```bash
$ terraform apply
 
Error: "ip_address_details": this field cannot be set
 
  on network.tf line 77, in resource "oci_load_balancer_load_balancer" "lb01":
  77: resource "oci_load_balancer_load_balancer" "lb01" {
```

公式サイトにTouble Shooting方法が書かれている。

[Troubleshooting | Guides | hashicorp/oci | Terraform Registry](https://registry.terraform.io/providers/hashicorp/oci/latest/docs/guides/troubleshooting#error-message-when-field-cannot-be-set)

上記によると、Terraformの**バージョンが古い時に発生する**とのことだった。

```bash
$ terraform --version
Terraform v0.12.36
```

`brew upgrade terraform`コマンドでアップグレードした後に、`terraform init`を実行すると違うエラーになった。

```bash
$ terraform --version
Terraform v0.14.5
 
$ terraform init
 
Initializing the backend...
 
Error: Invalid legacy provider address
 
This configuration or its associated state refers to the unqualified provider
"oci".
 
You must complete the Terraform 0.13 upgrade process before upgrading to later
versions.
```

providerのregistryが古いというエラーが出た。

## providerの更新

[Upgrading to Terraform v0.13 - Terraform by HashiCorp](https://www.terraform.io/upgrade-guides/0-13.html)

使用していたproviderは以下の通り

```bash
$ terraform providers
 
Providers required by configuration:
.
└── provider[registry.terraform.io/hashicorp/oci]
 
Providers required by state:
 
    provider[registry.terraform.io/-/oci]
```

Providerを更新してみる

```bash
$ terraform state replace-provider -- -/oci registry.terraform.io/hashicorp/oci
Terraform will perform the following actions:
 
  ~ Updating provider:
    - registry.terraform.io/-/oci
    + registry.terraform.io/hashicorp/oci
 
Changing 10 resources:
 
  oci_core_route_table.rt01
  oci_core_security_list.sl_web
  data.oci_identity_availability_domains.ads
  oci_core_instance.ubuntu02
  data.oci_core_public_ip.public_ip01
  oci_core_internet_gateway.ig01
  oci_core_instance.ubuntu01
  oci_core_subnet.subnet01
  data.oci_load_balancer_load_balancers.lb_test
  oci_core_virtual_network.vcn01
 
Do you want to make these changes?
Only 'yes' will be accepted to continue.
 
Enter a value: <yesを入力>
 
Successfully replaced provider for 10 resources.
```

無事に更新されたようで、これで`terraform init`がうまく行くようになった。

```bash
$ terraform providers
 
Providers required by configuration:
.
└── provider[registry.terraform.io/hashicorp/oci]
 
Providers required by state:
 
    provider[registry.terraform.io/hashicorp/oci]
```

しかし、terraform cloud上で`terraform init`する時にエラーとなった。
ローカルで発生した時と同じエラーだった。

```bash
Terraform v0.14.5
Configuring remote state backend...
Initializing Terraform configuration...
 
Setup failed: Failed terraform init (exit 1): <nil>
 
Output:
 
Initializing the backend...
 
Successfully configured the backend "remote"! Terraform will automatically
use this backend unless the backend configuration changes.
 
Error: Invalid legacy provider address
 
This configuration or its associated state refers to the unqualified provider
"oci".
 
You must complete the Terraform 0.13 upgrade process before upgrading to later
versions.
```

## tfstateファイルのversion up

[terraform Error: Invalid legacy provider address](https://blog.n-t.jp/tech/terraform-error-invalid-legacy-provider-address/)

エラーの原因を調べてみると、
terraformと`tfstate`ファイルのversionが違う時に発生する事がわかった。
terraformのversionが「0.14.X」だが、tfstateファイルのversionが「0.12.X」になっていた事が原因だった。

```tfstate:terraform.tfstate
{
  "version": 4,
  "terraform_version": "0.12.4",
  "serial": 27,

...
```

ローカル環境ではProviderを更新したタイミングで、`tfstate`ファイルも更新されていた。
実際に、providerを更新した時に、旧versionの`tfstate.backup`ファイルが作成された。

しかしながら、Terraform Cloudではterraformのversionは指定できるが、`tfstate`のversion更新させることはできない。
一度0.13.Xバージョンでで`terraform apply`を実行し、`tfstate`ファイルを手動でversion upさせる必要がある。

Web上のQueueから再実行できるが、`terraform plan`で変更がない場合は`terraform apply`は実行されず、`tfstate`ファイルも更新されなかった。

### Terraform Cloudのtfstateファイル更新

アナログなやり方だが、現状これ以外の方法は不明

1. Terraform Cloudにログインして、「Workspace」に移動する
2. 「Settings」 > 「General」より「0.13.X」にする
3. GitHubのmaster branchに、`tfstate`が変更されるように適当なcommitをする
4. Terraform Cloudで`terraform apply`が実行される
5. `tfstate`ファイルも「0.13.X」になる

0.14.Xまでversion upする時も、これと同じことを実施すれば`tfstate`ファイルが更新される。

