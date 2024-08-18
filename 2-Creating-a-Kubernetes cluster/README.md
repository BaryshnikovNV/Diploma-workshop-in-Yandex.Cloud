# Создание Kubernetes кластера
<details>
	<summary></summary>
      <br>

На этом этапе необходимо создать [Kubernetes](https://kubernetes.io/ru/docs/concepts/overview/what-is-kubernetes/) кластер на базе предварительно созданной инфраструктуры.   Требуется обеспечить доступ к ресурсам из Интернета.

Это можно сделать двумя способами:

1. Рекомендуемый вариант: самостоятельная установка Kubernetes кластера.  
   а. При помощи Terraform подготовить как минимум 3 виртуальных машины Compute Cloud для создания Kubernetes-кластера. Тип виртуальной машины следует выбрать самостоятельно с учётом требовании к производительности и стоимости. Если в дальнейшем поймете, что необходимо сменить тип инстанса, используйте Terraform для внесения изменений.  
   б. Подготовить [ansible](https://www.ansible.com/) конфигурации, можно воспользоваться, например [Kubespray](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/)  
   в. Задеплоить Kubernetes на подготовленные ранее инстансы, в случае нехватки каких-либо ресурсов вы всегда можете создать их при помощи Terraform.
2. Альтернативный вариант: воспользуйтесь сервисом [Yandex Managed Service for Kubernetes](https://cloud.yandex.ru/services/managed-kubernetes)  
  а. С помощью terraform resource для [kubernetes](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kubernetes_cluster) создать **региональный** мастер kubernetes с размещением нод в разных 3 подсетях      
  б. С помощью terraform resource для [kubernetes node group](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/kubernetes_node_group)
  
Ожидаемый результат:

1. Работоспособный Kubernetes кластер.
2. В файле `~/.kube/config` находятся данные для доступа к кластеру.
3. Команда `kubectl get pods --all-namespaces` отрабатывает без ошибок.

</details>

---
## Решение:

На этом этапе создадим `Kubernetes` кластер на базе предварительно созданной инфраструктуры.

### 2.1. При помощи [Terraform](./config/terraform) подготовим 3 виртуальных машины Compute Cloud для создания Kubernetes-кластера: 1 master-node и 2 worker-node.
<details>
	<summary></summary>
      <br>

```bash
baryshnikov@Huawei-PC:~/terraform$ terraform apply
data.yandex_compute_image.debian: Reading...
data.yandex_compute_image.debian: Read complete after 2s [id=fd8q49fvba72foa1ol22]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # local_file.hosts_yml_kubespray will be created
  + resource "local_file" "hosts_yml_kubespray" {
      + content              = (known after apply)
      + content_base64sha256 = (known after apply)
      + content_base64sha512 = (known after apply)
      + content_md5          = (known after apply)
      + content_sha1         = (known after apply)
      + content_sha256       = (known after apply)
      + content_sha512       = (known after apply)
      + directory_permission = "0777"
      + file_permission      = "0777"
      + filename             = "../ansible/inventory/hosts.yml"
      + id                   = (known after apply)
    }

  # yandex_compute_instance.master-node[0] will be created
  + resource "yandex_compute_instance" "master-node" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hostname                  = (known after apply)
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                debian:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCZteHhbi1uj16AhOsiZ/o/d/sqn+XEPmi9uaMcTiA/x7F7guE2QijmJHIQwF9GjhA4VXP3Oe/bWUHaORtSPFdLRkF39d7oOj08S48mIlighhGSL6ETK3l5z5m0Qv93nReqg//mOV3axleX09DIaiRubBAjG1Nzpa+7xyPHTHgfHXrV06yOF25lFmWHQs8smGnYuZDNQDQ1J76lR0cDTviWJRa2459cHWrA6m8Cu0qcLKTrwpU8DRWjhbRiIFWX8b2l4ZjfMC4LL67Gl/Ya2bhXpL5+sfEp5xakHTyTctd7tcJpZvjs4cwzHuW9d/Tl2SUjiaYRBtpERy8OwtRLVHITU7iKcHFwZEn4PEOngoF8QZXxeQPe8FbiVxzIoKcnvOMpW9UrFn1a0lTsPowB2t4x8ixNTCqWciV0eLzzs8UAbyZaDwlAz2okEE1g3evxPQJx80+tHRnTQtoBj2I4UydzRm7tYiP68XSZ+Zfxb605nRohLKsMwweP32FfR/iMNks= baryshnikov@Huawei-PC
            EOT
        }
      + name                      = "master-1"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd8q49fvba72foa1ol22"
              + name        = (known after apply)
              + size        = 30
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 4
          + memory        = 4
        }

      + scheduling_policy {
          + preemptible = true
        }
    }

  # yandex_compute_instance.worker-node[0] will be created
  + resource "yandex_compute_instance" "worker-node" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hostname                  = (known after apply)
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                debian:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCZteHhbi1uj16AhOsiZ/o/d/sqn+XEPmi9uaMcTiA/x7F7guE2QijmJHIQwF9GjhA4VXP3Oe/bWUHaORtSPFdLRkF39d7oOj08S48mIlighhGSL6ETK3l5z5m0Qv93nReqg//mOV3axleX09DIaiRubBAjG1Nzpa+7xyPHTHgfHXrV06yOF25lFmWHQs8smGnYuZDNQDQ1J76lR0cDTviWJRa2459cHWrA6m8Cu0qcLKTrwpU8DRWjhbRiIFWX8b2l4ZjfMC4LL67Gl/Ya2bhXpL5+sfEp5xakHTyTctd7tcJpZvjs4cwzHuW9d/Tl2SUjiaYRBtpERy8OwtRLVHITU7iKcHFwZEn4PEOngoF8QZXxeQPe8FbiVxzIoKcnvOMpW9UrFn1a0lTsPowB2t4x8ixNTCqWciV0eLzzs8UAbyZaDwlAz2okEE1g3evxPQJx80+tHRnTQtoBj2I4UydzRm7tYiP68XSZ+Zfxb605nRohLKsMwweP32FfR/iMNks= baryshnikov@Huawei-PC
            EOT
        }
      + name                      = "worker-1"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd8q49fvba72foa1ol22"
              + name        = (known after apply)
              + size        = 30
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 4
          + memory        = 8
        }

      + scheduling_policy {
          + preemptible = true
        }
    }

  # yandex_compute_instance.worker-node[1] will be created
  + resource "yandex_compute_instance" "worker-node" {
      + created_at                = (known after apply)
      + folder_id                 = (known after apply)
      + fqdn                      = (known after apply)
      + gpu_cluster_id            = (known after apply)
      + hostname                  = (known after apply)
      + id                        = (known after apply)
      + maintenance_grace_period  = (known after apply)
      + maintenance_policy        = (known after apply)
      + metadata                  = {
          + "serial-port-enable" = "1"
          + "ssh-keys"           = <<-EOT
                debian:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCZteHhbi1uj16AhOsiZ/o/d/sqn+XEPmi9uaMcTiA/x7F7guE2QijmJHIQwF9GjhA4VXP3Oe/bWUHaORtSPFdLRkF39d7oOj08S48mIlighhGSL6ETK3l5z5m0Qv93nReqg//mOV3axleX09DIaiRubBAjG1Nzpa+7xyPHTHgfHXrV06yOF25lFmWHQs8smGnYuZDNQDQ1J76lR0cDTviWJRa2459cHWrA6m8Cu0qcLKTrwpU8DRWjhbRiIFWX8b2l4ZjfMC4LL67Gl/Ya2bhXpL5+sfEp5xakHTyTctd7tcJpZvjs4cwzHuW9d/Tl2SUjiaYRBtpERy8OwtRLVHITU7iKcHFwZEn4PEOngoF8QZXxeQPe8FbiVxzIoKcnvOMpW9UrFn1a0lTsPowB2t4x8ixNTCqWciV0eLzzs8UAbyZaDwlAz2okEE1g3evxPQJx80+tHRnTQtoBj2I4UydzRm7tYiP68XSZ+Zfxb605nRohLKsMwweP32FfR/iMNks= baryshnikov@Huawei-PC
            EOT
        }
      + name                      = "worker-2"
      + network_acceleration_type = "standard"
      + platform_id               = "standard-v1"
      + service_account_id        = (known after apply)
      + status                    = (known after apply)
      + zone                      = (known after apply)

      + boot_disk {
          + auto_delete = true
          + device_name = (known after apply)
          + disk_id     = (known after apply)
          + mode        = (known after apply)

          + initialize_params {
              + block_size  = (known after apply)
              + description = (known after apply)
              + image_id    = "fd8q49fvba72foa1ol22"
              + name        = (known after apply)
              + size        = 30
              + snapshot_id = (known after apply)
              + type        = "network-hdd"
            }
        }

      + network_interface {
          + index              = (known after apply)
          + ip_address         = (known after apply)
          + ipv4               = true
          + ipv6               = (known after apply)
          + ipv6_address       = (known after apply)
          + mac_address        = (known after apply)
          + nat                = true
          + nat_ip_address     = (known after apply)
          + nat_ip_version     = (known after apply)
          + security_group_ids = (known after apply)
          + subnet_id          = (known after apply)
        }

      + resources {
          + core_fraction = 100
          + cores         = 4
          + memory        = 8
        }

      + scheduling_policy {
          + preemptible = true
        }
    }

  # yandex_iam_service_account.service will be created
  + resource "yandex_iam_service_account" "service" {
      + created_at  = (known after apply)
      + description = "service account to manage VMs"
      + folder_id   = "b1g7ijqfg6qt035hpbg8"
      + id          = (known after apply)
      + name        = "baryshnikov-nv"
    }

  # yandex_iam_service_account_static_access_key.terraform_service_account_key will be created
  + resource "yandex_iam_service_account_static_access_key" "terraform_service_account_key" {
      + access_key                   = (known after apply)
      + created_at                   = (known after apply)
      + description                  = "static access key for object storage"
      + encrypted_secret_key         = (known after apply)
      + id                           = (known after apply)
      + key_fingerprint              = (known after apply)
      + output_to_lockbox_version_id = (known after apply)
      + secret_key                   = (sensitive value)
      + service_account_id           = (known after apply)
    }

  # yandex_resourcemanager_folder_iam_member.editor will be created
  + resource "yandex_resourcemanager_folder_iam_member" "editor" {
      + folder_id = "b1g7ijqfg6qt035hpbg8"
      + id        = (known after apply)
      + member    = (known after apply)
      + role      = "editor"
    }

  # yandex_storage_bucket.state_storage will be created
  + resource "yandex_storage_bucket" "state_storage" {
      + access_key            = (known after apply)
      + bucket                = (known after apply)
      + bucket_domain_name    = (known after apply)
      + default_storage_class = (known after apply)
      + folder_id             = (known after apply)
      + force_destroy         = false
      + id                    = (known after apply)
      + secret_key            = (sensitive value)
      + website_domain        = (known after apply)
      + website_endpoint      = (known after apply)

      + anonymous_access_flags {
          + list = false
          + read = false
        }
    }

  # yandex_storage_object.backend will be created
  + resource "yandex_storage_object" "backend" {
      + access_key   = (known after apply)
      + acl          = "private"
      + bucket       = (known after apply)
      + content_type = (known after apply)
      + id           = (known after apply)
      + key          = "terraform.tfstate"
      + secret_key   = (sensitive value)
      + source       = "./terraform.tfstate"
    }

  # yandex_vpc_network.vpc0 will be created
  + resource "yandex_vpc_network" "vpc0" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = "vpc0"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_subnet.subnet-a will be created
  + resource "yandex_vpc_subnet" "subnet-a" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "subnet-a"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "10.0.1.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

  # yandex_vpc_subnet.subnet-b will be created
  + resource "yandex_vpc_subnet" "subnet-b" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "subnet-b"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "10.0.2.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-b"
    }

  # yandex_vpc_subnet.subnet-d will be created
  + resource "yandex_vpc_subnet" "subnet-d" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "subnet-d"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "10.0.3.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-d"
    }

Plan: 13 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + master-node = [
      + {
          + ip_external = (known after apply)
          + ip_internal = (known after apply)
          + name        = "master-1"
        },
    ]
  + worker-node = [
      + {
          + ip_external = (known after apply)
          + ip_internal = (known after apply)
          + name        = "worker-1"
        },
      + {
          + ip_external = (known after apply)
          + ip_internal = (known after apply)
          + name        = "worker-2"
        },
    ]

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

yandex_iam_service_account.service: Creating...
yandex_vpc_network.vpc0: Creating...
yandex_iam_service_account.service: Creation complete after 3s [id=ajeovc7kmove0560uav8]
yandex_resourcemanager_folder_iam_member.editor: Creating...
yandex_iam_service_account_static_access_key.terraform_service_account_key: Creating...
yandex_vpc_network.vpc0: Creation complete after 3s [id=enp9vmcj3oment5ce6cs]
yandex_vpc_subnet.subnet-a: Creating...
yandex_vpc_subnet.subnet-b: Creating...
yandex_vpc_subnet.subnet-d: Creating...
yandex_vpc_subnet.subnet-b: Creation complete after 1s [id=e2lgsooc27cj4q8pd6cl]
yandex_vpc_subnet.subnet-a: Creation complete after 1s [id=e9bvlvbjqi56mg1n94nt]
yandex_compute_instance.worker-node[0]: Creating...
yandex_compute_instance.worker-node[1]: Creating...
yandex_compute_instance.master-node[0]: Creating...
yandex_iam_service_account_static_access_key.terraform_service_account_key: Creation complete after 1s [id=ajel2g9qdb8tqc8idm0a]
yandex_storage_bucket.state_storage: Creating...
yandex_vpc_subnet.subnet-d: Creation complete after 2s [id=fl8f4b605oq8bdv8de95]
yandex_resourcemanager_folder_iam_member.editor: Creation complete after 4s [id=b1g7ijqfg6qt035hpbg8/editor/serviceAccount:ajeovc7kmove0560uav8]
yandex_storage_bucket.state_storage: Creation complete after 6s [id=state-storage-18-08-2024]
yandex_storage_object.backend: Creating...
yandex_storage_object.backend: Creation complete after 0s [id=terraform.tfstate]
yandex_compute_instance.master-node[0]: Still creating... [10s elapsed]
yandex_compute_instance.worker-node[1]: Still creating... [10s elapsed]
yandex_compute_instance.worker-node[0]: Still creating... [10s elapsed]
yandex_compute_instance.worker-node[0]: Still creating... [20s elapsed]
yandex_compute_instance.worker-node[1]: Still creating... [20s elapsed]
yandex_compute_instance.master-node[0]: Still creating... [20s elapsed]
yandex_compute_instance.master-node[0]: Still creating... [30s elapsed]
yandex_compute_instance.worker-node[1]: Still creating... [30s elapsed]
yandex_compute_instance.worker-node[0]: Still creating... [30s elapsed]
yandex_compute_instance.worker-node[1]: Creation complete after 36s [id=fhmm5drqq47fkfo9878k]
yandex_compute_instance.worker-node[0]: Still creating... [40s elapsed]
yandex_compute_instance.master-node[0]: Still creating... [40s elapsed]
yandex_compute_instance.worker-node[0]: Creation complete after 44s [id=fhmit68a5erin5ldu9hn]
yandex_compute_instance.master-node[0]: Creation complete after 46s [id=fhmbb7q6q8m12ktto9qc]
local_file.hosts_yml_kubespray: Creating...
local_file.hosts_yml_kubespray: Creation complete after 0s [id=f56c609400a39e98b58436c056c218ebabce8092]

Apply complete! Resources: 13 added, 0 changed, 0 destroyed.

Outputs:

master-node = [
  {
    "ip_external" = "89.169.149.215"
    "ip_internal" = "10.0.1.35"
    "name" = "master-1"
  },
]
worker-node = [
  {
    "ip_external" = "89.169.130.108"
    "ip_internal" = "10.0.1.8"
    "name" = "worker-1"
  },
  {
    "ip_external" = "89.169.138.181"
    "ip_internal" = "10.0.1.11"
    "name" = "worker-2"
  },
]
```

</details>

Проверим созданные ресурсы с помощью CLI:
```bash
baryshnikov@Huawei-PC:~/terraform$ yc vpc network list
+----------------------+------+
|          ID          | NAME |
+----------------------+------+
| enp9vmcj3oment5ce6cs | vpc0 |
+----------------------+------+

baryshnikov@Huawei-PC:~/terraform$ yc vpc subnet list
+----------------------+----------+----------------------+----------------+---------------+---------------+
|          ID          |   NAME   |      NETWORK ID      | ROUTE TABLE ID |     ZONE      |     RANGE     |
+----------------------+----------+----------------------+----------------+---------------+---------------+
| e2lgsooc27cj4q8pd6cl | subnet-b | enp9vmcj3oment5ce6cs |                | ru-central1-b | [10.0.2.0/24] |
| e9bvlvbjqi56mg1n94nt | subnet-a | enp9vmcj3oment5ce6cs |                | ru-central1-a | [10.0.1.0/24] |
| fl8f4b605oq8bdv8de95 | subnet-d | enp9vmcj3oment5ce6cs |                | ru-central1-d | [10.0.3.0/24] |
+----------------------+----------+----------------------+----------------+---------------+---------------+

baryshnikov@Huawei-PC:~/terraform$ yc compute instance list
+----------------------+----------+---------------+---------+----------------+-------------+
|          ID          |   NAME   |    ZONE ID    | STATUS  |  EXTERNAL IP   | INTERNAL IP |
+----------------------+----------+---------------+---------+----------------+-------------+
| fhmbb7q6q8m12ktto9qc | master-1 | ru-central1-a | RUNNING | 89.169.149.215 | 10.0.1.35   |
| fhmit68a5erin5ldu9hn | worker-1 | ru-central1-a | RUNNING | 89.169.130.108 | 10.0.1.8    |
| fhmm5drqq47fkfo9878k | worker-2 | ru-central1-a | RUNNING | 89.169.138.181 | 10.0.1.11   |
+----------------------+----------+---------------+---------+----------------+-------------+

baryshnikov@Huawei-PC:~/terraform$ yc storage bucket list
+--------------------------+----------------------+----------+-----------------------+---------------------+
|           NAME           |      FOLDER ID       | MAX SIZE | DEFAULT STORAGE CLASS |     CREATED AT      |
+--------------------------+----------------------+----------+-----------------------+---------------------+
| state-storage-18-08-2024 | b1g7ijqfg6qt035hpbg8 |        0 | STANDARD              | 2024-08-18 07:56:37 |
+--------------------------+----------------------+----------+-----------------------+---------------------+

baryshnikov@Huawei-PC:~/terraform$ yc storage bucket stats --name state-storage-18-08-2024
name: state-storage-18-08-2024
default_storage_class: STANDARD
anonymous_access_flags:
  read: false
  list: false
  config_read: false
created_at: "2024-08-18T07:56:37.693951Z"
updated_at: "2024-08-18T07:56:37.693951Z"
```

Так же, помимо создание ВМ, сделаем с помощью terraform генерацию файлика `hosts.yml` с разу в папку с ansible для последующего cоздания kubernetes-кластера при помощи ansible-playbook kubsprey.
Файл hosts.yml
```yml
all:
  hosts:
    master-1:
      ansible_host: 89.169.149.215
      ip: 10.0.1.35
      access_ip: 10.0.1.35
      ansible_user: debian
    worker-1:
      ansible_host: 89.169.130.108
      ip: 10.0.1.8
      access_ip: 10.0.1.8
      ansible_user: debian
    worker-2:
      ansible_host: 89.169.138.181
      ip: 10.0.1.11
      access_ip: 10.0.1.11
      ansible_user: debian
  children:
    kube_control_plane:
      hosts:
        master-1:
    kube_node:
      hosts:
        worker-1:
        worker-2:
    etcd:
      hosts:
        master-1:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```

---
### 2.2. Подготовим [ansible конфигурацию](./config/ansible) для установки kubspray.

Запустим выполнение ansible-playbook на master-node, который скачает `kubspray`, установит все необходимые для него зависимости из файла `requirements.txt` и скопирует на мастер приватный ключ. 

<details>
	<summary></summary>
      <br>

```
baryshnikov@Huawei-PC:~/ansible$ ansible-playbook -i inventory/hosts.yml site.yml

PLAY [Установка pip] *******************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
The authenticity of host '89.169.149.215 (89.169.149.215)' can't be established.
ED25519 key fingerprint is SHA256:UVx345/H8NnUP7e6uU5q0xb+pFc1gV7xY+FP9yU0K/w.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
ok: [master-1]

TASK [Скачиваем файл get-pip.py] *******************************************************************************************************************************
changed: [master-1]

TASK [Удаляем EXTERNALLY-MANAGED] ******************************************************************************************************************************
changed: [master-1]

TASK [Устанавливаем pip] ***************************************************************************************************************************************
changed: [master-1]

PLAY [Установка зависимостей из ansible-playbook kubespray] ****************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
ok: [master-1]

TASK [Выполнение apt update и установка git] *******************************************************************************************************************
changed: [master-1]

TASK [Клонируем kubespray из репозитория] **********************************************************************************************************************
changed: [master-1]

TASK [Изменение прав на папку kubspray] ************************************************************************************************************************
changed: [master-1]

TASK [Установка зависимостей из requirements.txt] **************************************************************************************************************
changed: [master-1]

TASK [Копирование содержимого папки inventory/sample в папку inventory/mycluster] ******************************************************************************
changed: [master-1]

PLAY [Подготовка master-node к установке kubespray из ansible-playbook] ****************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
ok: [master-1]

TASK [Копирование на master-node файла hosts.yml] **************************************************************************************************************
changed: [master-1]

TASK [Копирование на мастер приватного ключа] ******************************************************************************************************************
changed: [master-1]

PLAY RECAP *****************************************************************************************************************************************************
master-1                   : ok=13   changed=10   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

</details>

Далее зайдем на master-node и запустим ansible-playbook kubspray для установки кластера kubernetes с помощью следующей команды.
```bash
ansible-playbook -i inventory/mycluster/hosts.yml cluster.yml -b -v
```

Результат выполнения playbook:
```bash
PLAY RECAP *****************************************************************************************************************************************************
master-1                   : ok=606  changed=42   unreachable=0    failed=0    skipped=1117 rescued=0    ignored=5   
worker-1                   : ok=404  changed=14   unreachable=0    failed=0    skipped=671  rescued=0    ignored=1   
worker-2                   : ok=404  changed=14   unreachable=0    failed=0    skipped=667  rescued=0    ignored=1   

Sunday 18 August 2024  08:22:16 +0000 (0:00:00.131)       0:07:19.051 ********* 
=============================================================================== 
kubernetes/control-plane : Kubeadm | Initialize first master ------------------------------------------------------------------------------------------- 67.80s
kubernetes/kubeadm : Join to cluster ------------------------------------------------------------------------------------------------------------------- 21.51s
container-engine/containerd : Download_file | Download item --------------------------------------------------------------------------------------------- 5.30s
container-engine/crictl : Download_file | Download item ------------------------------------------------------------------------------------------------- 5.23s
container-engine/runc : Download_file | Download item --------------------------------------------------------------------------------------------------- 5.16s
container-engine/nerdctl : Download_file | Download item ------------------------------------------------------------------------------------------------ 5.12s
kubernetes-apps/ansible : Kubernetes Apps | Lay Down CoreDNS templates ---------------------------------------------------------------------------------- 4.76s
kubernetes-apps/ansible : Kubernetes Apps | Start Resources --------------------------------------------------------------------------------------------- 4.59s
container-engine/crictl : Extract_file | Unpacking archive ---------------------------------------------------------------------------------------------- 4.31s
network_plugin/calico : Wait for calico kubeconfig to be created ---------------------------------------------------------------------------------------- 4.29s
container-engine/nerdctl : Extract_file | Unpacking archive --------------------------------------------------------------------------------------------- 4.17s
container-engine/containerd : Containerd | Unpack containerd archive ------------------------------------------------------------------------------------ 3.95s
network_plugin/cni : CNI | Copy cni plugins ------------------------------------------------------------------------------------------------------------- 3.79s
download : Download_file | Download item ---------------------------------------------------------------------------------------------------------------- 3.55s
etcdctl_etcdutl : Download_file | Download item --------------------------------------------------------------------------------------------------------- 3.45s
etcdctl_etcdutl : Extract_file | Unpacking archive ------------------------------------------------------------------------------------------------------ 3.30s
container-engine/validate-container-engine : Populate service facts ------------------------------------------------------------------------------------- 3.00s
network_plugin/calico : Calico | Create calico manifests ------------------------------------------------------------------------------------------------ 2.76s
network_plugin/calico : Calico | Configure calico FelixConfiguration ------------------------------------------------------------------------------------ 2.28s
etcdctl_etcdutl : Copy etcd binary ---------------------------------------------------------------------------------------------------------------------- 2.16s
```

Для выполнения команд kubectl без sudo скопируем папку `.kube` в домашнюю дирректорию пользователя и сменим владельца, а также группу владельцев папки с файлами:
```bash
debian@fhmpu5s2lo5h1d9ka5am:~/kubespray$ sudo cp -r /root/.kube ~/
debian@fhmpu5s2lo5h1d9ka5am:~/kubespray$ 
debian@fhmpu5s2lo5h1d9ka5am:~/kubespray$
debian@fhmpu5s2lo5h1d9ka5am:~/kubespray$ sudo chown -R debian:debian ~/.kube
```

Проверим работоспособность кластера:
```bash
debian@fhmbb7q6q8m12ktto9qc:~/kubespray$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY   STATUS    RESTARTS       AGE
kube-system   calico-kube-controllers-c7cc688f8-f6ldp   1/1     Running   0              114s
kube-system   calico-node-6vhk8                         1/1     Running   0              2m13s
kube-system   calico-node-cfm7w                         1/1     Running   0              2m13s
kube-system   calico-node-n6gwb                         1/1     Running   0              2m13s
kube-system   coredns-776bb9db5d-82v8f                  1/1     Running   0              103s
kube-system   coredns-776bb9db5d-r6zkg                  1/1     Running   0              86s
kube-system   dns-autoscaler-6ffb84bd6-5t5f8            1/1     Running   0              101s
kube-system   kube-apiserver-master-1                   1/1     Running   0              3m15s
kube-system   kube-controller-manager-master-1          1/1     Running   1              3m15s
kube-system   kube-proxy-b99ts                          1/1     Running   0              2m34s
kube-system   kube-proxy-cddk2                          1/1     Running   0              2m34s
kube-system   kube-proxy-r5gk2                          1/1     Running   0              2m34s
kube-system   kube-scheduler-master-1                   1/1     Running   1              3m15s
kube-system   nginx-proxy-worker-1                      1/1     Running   0              2m36s
kube-system   nginx-proxy-worker-2                      1/1     Running   0              2m31s
kube-system   nodelocaldns-2c62k                        1/1     Running   1 (100s ago)   100s
kube-system   nodelocaldns-f2ghg                        1/1     Running   4 (56s ago)    100s
kube-system   nodelocaldns-stfpr                        1/1     Running   1 (100s ago)   100s
```
