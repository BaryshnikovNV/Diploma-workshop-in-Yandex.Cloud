# Создание облачной инфраструктуры
<details>
	<summary></summary>
      <br>

Для начала необходимо подготовить облачную инфраструктуру в ЯО при помощи [Terraform](https://www.terraform.io/).

Особенности выполнения:

- Бюджет купона ограничен, что следует иметь в виду при проектировании инфраструктуры и использовании ресурсов;
Для облачного k8s используйте региональный мастер(неотказоустойчивый). Для self-hosted k8s минимизируйте ресурсы ВМ и долю ЦПУ. В обоих вариантах используйте прерываемые ВМ для worker nodes.

Предварительная подготовка к установке и запуску Kubernetes кластера.

1. Создайте сервисный аккаунт, который будет в дальнейшем использоваться Terraform для работы с инфраструктурой с необходимыми и достаточными правами. Не стоит использовать права суперпользователя
2. Подготовьте [backend](https://www.terraform.io/docs/language/settings/backends/index.html) для Terraform:  
   а. Рекомендуемый вариант: S3 bucket в созданном ЯО аккаунте(создание бакета через TF)
   б. Альтернативный вариант:  [Terraform Cloud](https://app.terraform.io/)  
3. Создайте VPC с подсетями в разных зонах доступности.
4. Убедитесь, что теперь вы можете выполнить команды `terraform destroy` и `terraform apply` без дополнительных ручных действий.
5. В случае использования [Terraform Cloud](https://app.terraform.io/) в качестве [backend](https://www.terraform.io/docs/language/settings/backends/index.html) убедитесь, что применение изменений успешно проходит, используя web-интерфейс Terraform cloud.

Ожидаемые результаты:

1. Terraform сконфигурирован и создание инфраструктуры посредством Terraform возможно без дополнительных ручных действий.
2. Полученная конфигурация инфраструктуры является предварительной, поэтому в ходе дальнейшего выполнения задания возможны изменения.

</details>

---
## Решение:

Подготовим облачную инфраструктуру в Яндекс.Облако при помощи Terraform.

### 1.1. Создадим сервисный аккаунт, который будет в дальнейшем использоваться Terraform для работы с инфраструктурой с необходимыми и достаточными правами.

```hcl
# Создание сервисного аккаунта для Terraform
resource "yandex_iam_service_account" "service" {
  name      = var.account_name
  description = "service account to manage VMs"
  folder_id = var.folder_id
}

# Назначение роли editor сервисному аккаунту
resource "yandex_resourcemanager_folder_iam_member" "editor" {
  folder_id = var.folder_id
  role      = "editor"
  member    = "serviceAccount:${yandex_iam_service_account.service.id}"
  depends_on = [yandex_iam_service_account.service]
}

# Создание статического ключа доступа для сервисного аккаунта
resource "yandex_iam_service_account_static_access_key" "terraform_service_account_key" {
  service_account_id = yandex_iam_service_account.service.id
  description        = "static access key for object storage"
}
```

---
### 1.2. Подготовим backend для Terraform:  

```hcl
# Создадим бакет с использованием ключа
resource "yandex_storage_bucket" "state_storage" {
  bucket     = local.bucket_name
  access_key = yandex_iam_service_account_static_access_key.terraform_service_account_key.access_key
  secret_key = yandex_iam_service_account_static_access_key.terraform_service_account_key.secret_key

  anonymous_access_flags {
    read = false
    list = false
  }
}

# Локальная переменная отвечающая за текущую дату в названии бакета
locals {
    current_timestamp = timestamp()
    formatted_date = formatdate("DD-MM-YYYY", local.current_timestamp)
    bucket_name = "state-storage-${local.formatted_date}"
}

# Создание объекта в существующей папке
resource "yandex_storage_object" "backend" {
  access_key = yandex_iam_service_account_static_access_key.terraform_service_account_key.access_key
  secret_key = yandex_iam_service_account_static_access_key.terraform_service_account_key.secret_key
  bucket = local.bucket_name
  key    = "terraform.tfstate"
  source = "./terraform.tfstate"
  depends_on = [yandex_storage_bucket.state_storage]
}
```

---
### 1.3. Создадим VPC с подсетями в разных зонах доступности.

```hcl
#Создание пустой VPC
resource "yandex_vpc_network" "vpc0" {
  name = var.vpc_name
}

#Создадим в VPC subnet c названием subnet-a
resource "yandex_vpc_subnet" "subnet-a" {
  name           = var.subnet-a
  zone           = var.zone-a
  network_id     = yandex_vpc_network.vpc0.id
  v4_cidr_blocks = var.cidr-a
}

#Создание в VPC subnet с названием subnet-b
resource "yandex_vpc_subnet" "subnet-b" {
  name           = var.subnet-b
  zone           = var.zone-b
  network_id     = yandex_vpc_network.vpc0.id
  v4_cidr_blocks = var.cidr-b
}

#Создание в VPC subnet с названием subnet-d
resource "yandex_vpc_subnet" "subnet-d" {
  name           = var.subnet-d
  zone           = var.zone-d
  network_id     = yandex_vpc_network.vpc0.id
  v4_cidr_blocks = var.cidr-d
}



variable "vpc_name" {
  type        = string
  default     = "vpc0"
  description = "VPC network"
}

variable "subnet-a" {
  type        = string
  default     = "subnet-a"
  description = "subnet name"
}

variable "subnet-b" {
  type        = string
  default     = "subnet-b"
  description = "subnet name"
}

variable "subnet-d" {
  type        = string
  default     = "subnet-d"
  description = "subnet name"
}

variable "zone-a" {
  type        = string
  default     = "ru-central1-a"
  description = "https://cloud.yandex.ru/docs/overview/concepts/geo-scope"
}

variable "zone-b" {
  type        = string
  default     = "ru-central1-b"
  description = "https://cloud.yandex.ru/docs/overview/concepts/geo-scope"
}

variable "zone-d" {
  type        = string
  default     = "ru-central1-d"
  description = "https://cloud.yandex.ru/docs/overview/concepts/geo-scope"
}

variable "cidr-a" {
  type        = list(string)
  default     = ["10.0.1.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}

variable "cidr-b" {
  type        = list(string)
  default     = ["10.0.2.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}

variable "cidr-d" {
  type        = list(string)
  default     = ["10.0.3.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}
```

---
### 1.4. Убедимся, что теперь выполняется команды `terraform apply` без дополнительных ручных действий.
<details>
	<summary></summary>
      <br>

```bash
baryshnikov@Huawei-PC:~/terraform$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

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

Plan: 9 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

yandex_iam_service_account.service: Creating...
yandex_vpc_network.vpc0: Creating...
yandex_vpc_network.vpc0: Creation complete after 2s [id=enpatvrb6a0r8p2eok1s]
yandex_vpc_subnet.subnet-d: Creating...
yandex_vpc_subnet.subnet-b: Creating...
yandex_vpc_subnet.subnet-a: Creating...
yandex_vpc_subnet.subnet-a: Creation complete after 1s [id=e9bb7bh3ut5e4qvkp870]
yandex_iam_service_account.service: Creation complete after 3s [id=aje0chuggmt91h8iu8qj]
yandex_resourcemanager_folder_iam_member.editor: Creating...
yandex_iam_service_account_static_access_key.terraform_service_account_key: Creating...
yandex_vpc_subnet.subnet-d: Creation complete after 1s [id=fl8deoov076qfv0mkg9r]
yandex_vpc_subnet.subnet-b: Creation complete after 2s [id=e2ln3l9s6udpnviq1n72]
yandex_iam_service_account_static_access_key.terraform_service_account_key: Creation complete after 1s [id=ajekk82rdl8vimpnhhqk]
yandex_storage_bucket.state_storage: Creating...
yandex_resourcemanager_folder_iam_member.editor: Creation complete after 5s [id=b1g7ijqfg6qt035hpbg8/editor/serviceAccount:aje0chuggmt91h8iu8qj]
yandex_storage_bucket.state_storage: Creation complete after 7s [id=state-storage-17-08-2024]
yandex_storage_object.backend: Creating...
yandex_storage_object.backend: Creation complete after 0s [id=terraform.tfstate]

Apply complete! Resources: 9 added, 0 changed, 0 destroyed.
```

</details>

Посмотрим созданные ресурсы с помощью CLI

```bash
baryshnikov@Huawei-PC:~/terraform$ yc vpc network list
+----------------------+------+
|          ID          | NAME |
+----------------------+------+
| enpatvrb6a0r8p2eok1s | vpc0 |
+----------------------+------+

baryshnikov@Huawei-PC:~/terraform$ yc vpc subnet list
+----------------------+----------+----------------------+----------------+---------------+---------------+
|          ID          |   NAME   |      NETWORK ID      | ROUTE TABLE ID |     ZONE      |     RANGE     |
+----------------------+----------+----------------------+----------------+---------------+---------------+
| e2ln3l9s6udpnviq1n72 | subnet-b | enpatvrb6a0r8p2eok1s |                | ru-central1-b | [10.0.2.0/24] |
| e9bb7bh3ut5e4qvkp870 | subnet-a | enpatvrb6a0r8p2eok1s |                | ru-central1-a | [10.0.1.0/24] |
| fl8deoov076qfv0mkg9r | subnet-d | enpatvrb6a0r8p2eok1s |                | ru-central1-d | [10.0.3.0/24] |
+----------------------+----------+----------------------+----------------+---------------+---------------+

baryshnikov@Huawei-PC:~/terraform$ yc storage bucket list
+--------------------------+----------------------+----------+-----------------------+---------------------+
|           NAME           |      FOLDER ID       | MAX SIZE | DEFAULT STORAGE CLASS |     CREATED AT      |
+--------------------------+----------------------+----------+-----------------------+---------------------+
| state-storage-17-08-2024 | b1g7ijqfg6qt035hpbg8 |        0 | STANDARD              | 2024-08-17 07:18:25 |
+--------------------------+----------------------+----------+-----------------------+---------------------+

baryshnikov@Huawei-PC:~/terraform$ yc storage bucket stats --name state-storage-17-08-2024
name: state-storage-17-08-2024
default_storage_class: STANDARD
anonymous_access_flags:
  read: false
  list: false
  config_read: false
created_at: "2024-08-17T07:18:25.460399Z"
updated_at: "2024-08-17T07:18:25.460399Z"
```

---
### 1.5. Убедимся, что теперь выполняется команды `terraform destroy` без дополнительных ручных действий.
<details>
	<summary></summary>
      <br>

```bash
baryshnikov@Huawei-PC:~/terraform$ terraform destroy
yandex_iam_service_account.service: Refreshing state... [id=aje0chuggmt91h8iu8qj]
yandex_vpc_network.vpc0: Refreshing state... [id=enpatvrb6a0r8p2eok1s]
yandex_resourcemanager_folder_iam_member.editor: Refreshing state... [id=b1g7ijqfg6qt035hpbg8/editor/serviceAccount:aje0chuggmt91h8iu8qj]
yandex_iam_service_account_static_access_key.terraform_service_account_key: Refreshing state... [id=ajekk82rdl8vimpnhhqk]
yandex_vpc_subnet.subnet-b: Refreshing state... [id=e2ln3l9s6udpnviq1n72]
yandex_vpc_subnet.subnet-a: Refreshing state... [id=e9bb7bh3ut5e4qvkp870]
yandex_vpc_subnet.subnet-d: Refreshing state... [id=fl8deoov076qfv0mkg9r]
yandex_storage_bucket.state_storage: Refreshing state... [id=state-storage-17-08-2024]
yandex_storage_object.backend: Refreshing state... [id=terraform.tfstate]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # yandex_iam_service_account.service will be destroyed
  - resource "yandex_iam_service_account" "service" {
      - created_at  = "2024-08-17T07:18:21Z" -> null
      - description = "service account to manage VMs" -> null
      - folder_id   = "b1g7ijqfg6qt035hpbg8" -> null
      - id          = "aje0chuggmt91h8iu8qj" -> null
      - name        = "baryshnikov-nv" -> null
    }

  # yandex_iam_service_account_static_access_key.terraform_service_account_key will be destroyed
  - resource "yandex_iam_service_account_static_access_key" "terraform_service_account_key" {
      - access_key         = "YCAJEUVuiAAxSGo0kcx8i7yvg" -> null
      - created_at         = "2024-08-17T07:18:23Z" -> null
      - description        = "static access key for object storage" -> null
      - id                 = "ajekk82rdl8vimpnhhqk" -> null
      - secret_key         = (sensitive value) -> null
      - service_account_id = "aje0chuggmt91h8iu8qj" -> null
    }

  # yandex_resourcemanager_folder_iam_member.editor will be destroyed
  - resource "yandex_resourcemanager_folder_iam_member" "editor" {
      - folder_id = "b1g7ijqfg6qt035hpbg8" -> null
      - id        = "b1g7ijqfg6qt035hpbg8/editor/serviceAccount:aje0chuggmt91h8iu8qj" -> null
      - member    = "serviceAccount:aje0chuggmt91h8iu8qj" -> null
      - role      = "editor" -> null
    }

  # yandex_storage_bucket.state_storage will be destroyed
  - resource "yandex_storage_bucket" "state_storage" {
      - access_key            = "YCAJEUVuiAAxSGo0kcx8i7yvg" -> null
      - bucket                = "state-storage-17-08-2024" -> null
      - bucket_domain_name    = "state-storage-17-08-2024.storage.yandexcloud.net" -> null
      - default_storage_class = "STANDARD" -> null
      - folder_id             = "b1g7ijqfg6qt035hpbg8" -> null
      - force_destroy         = false -> null
      - id                    = "state-storage-17-08-2024" -> null
      - max_size              = 0 -> null
      - secret_key            = (sensitive value) -> null
      - tags                  = {} -> null

      - anonymous_access_flags {
          - config_read = false -> null
          - list        = false -> null
          - read        = false -> null
        }

      - versioning {
          - enabled = false -> null
        }
    }

  # yandex_storage_object.backend will be destroyed
  - resource "yandex_storage_object" "backend" {
      - access_key   = "YCAJEUVuiAAxSGo0kcx8i7yvg" -> null
      - acl          = "private" -> null
      - bucket       = "state-storage-17-08-2024" -> null
      - content_type = "application/octet-stream" -> null
      - id           = "terraform.tfstate" -> null
      - key          = "terraform.tfstate" -> null
      - secret_key   = (sensitive value) -> null
      - source       = "./terraform.tfstate" -> null
      - tags         = {} -> null
    }

  # yandex_vpc_network.vpc0 will be destroyed
  - resource "yandex_vpc_network" "vpc0" {
      - created_at                = "2024-08-17T07:18:21Z" -> null
      - default_security_group_id = "enp7haeedua1i6hbrgtr" -> null
      - folder_id                 = "b1g7ijqfg6qt035hpbg8" -> null
      - id                        = "enpatvrb6a0r8p2eok1s" -> null
      - labels                    = {} -> null
      - name                      = "vpc0" -> null
      - subnet_ids                = [
          - "e2ln3l9s6udpnviq1n72",
          - "e9bb7bh3ut5e4qvkp870",
          - "fl8deoov076qfv0mkg9r",
        ] -> null
    }

  # yandex_vpc_subnet.subnet-a will be destroyed
  - resource "yandex_vpc_subnet" "subnet-a" {
      - created_at     = "2024-08-17T07:18:23Z" -> null
      - folder_id      = "b1g7ijqfg6qt035hpbg8" -> null
      - id             = "e9bb7bh3ut5e4qvkp870" -> null
      - labels         = {} -> null
      - name           = "subnet-a" -> null
      - network_id     = "enpatvrb6a0r8p2eok1s" -> null
      - v4_cidr_blocks = [
          - "10.0.1.0/24",
        ] -> null
      - v6_cidr_blocks = [] -> null
      - zone           = "ru-central1-a" -> null
    }

  # yandex_vpc_subnet.subnet-b will be destroyed
  - resource "yandex_vpc_subnet" "subnet-b" {
      - created_at     = "2024-08-17T07:18:24Z" -> null
      - folder_id      = "b1g7ijqfg6qt035hpbg8" -> null
      - id             = "e2ln3l9s6udpnviq1n72" -> null
      - labels         = {} -> null
      - name           = "subnet-b" -> null
      - network_id     = "enpatvrb6a0r8p2eok1s" -> null
      - v4_cidr_blocks = [
          - "10.0.2.0/24",
        ] -> null
      - v6_cidr_blocks = [] -> null
      - zone           = "ru-central1-b" -> null
    }

  # yandex_vpc_subnet.subnet-d will be destroyed
  - resource "yandex_vpc_subnet" "subnet-d" {
      - created_at     = "2024-08-17T07:18:23Z" -> null
      - folder_id      = "b1g7ijqfg6qt035hpbg8" -> null
      - id             = "fl8deoov076qfv0mkg9r" -> null
      - labels         = {} -> null
      - name           = "subnet-d" -> null
      - network_id     = "enpatvrb6a0r8p2eok1s" -> null
      - v4_cidr_blocks = [
          - "10.0.3.0/24",
        ] -> null
      - v6_cidr_blocks = [] -> null
      - zone           = "ru-central1-d" -> null
    }

Plan: 0 to add, 0 to change, 9 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

yandex_resourcemanager_folder_iam_member.editor: Destroying... [id=b1g7ijqfg6qt035hpbg8/editor/serviceAccount:aje0chuggmt91h8iu8qj]
yandex_vpc_subnet.subnet-d: Destroying... [id=fl8deoov076qfv0mkg9r]
yandex_vpc_subnet.subnet-b: Destroying... [id=e2ln3l9s6udpnviq1n72]
yandex_vpc_subnet.subnet-a: Destroying... [id=e9bb7bh3ut5e4qvkp870]
yandex_storage_object.backend: Destroying... [id=terraform.tfstate]
yandex_storage_object.backend: Destruction complete after 1s
yandex_storage_bucket.state_storage: Destroying... [id=state-storage-17-08-2024]
yandex_vpc_subnet.subnet-d: Destruction complete after 1s
yandex_vpc_subnet.subnet-b: Destruction complete after 2s
yandex_vpc_subnet.subnet-a: Destruction complete after 3s
yandex_vpc_network.vpc0: Destroying... [id=enpatvrb6a0r8p2eok1s]
yandex_vpc_network.vpc0: Destruction complete after 0s
yandex_resourcemanager_folder_iam_member.editor: Destruction complete after 5s
yandex_storage_bucket.state_storage: Still destroying... [id=state-storage-17-08-2024, 10s elapsed]
yandex_storage_bucket.state_storage: Destruction complete after 11s
yandex_iam_service_account_static_access_key.terraform_service_account_key: Destroying... [id=ajekk82rdl8vimpnhhqk]
yandex_iam_service_account_static_access_key.terraform_service_account_key: Destruction complete after 1s
yandex_iam_service_account.service: Destroying... [id=aje0chuggmt91h8iu8qj]
yandex_iam_service_account.service: Destruction complete after 2s

Destroy complete! Resources: 9 destroyed.
```