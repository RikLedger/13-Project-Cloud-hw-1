# Домашнее задание к занятию «Организация сети» `Горбачев Олег`

### Задание 1. Yandex Cloud 

**Что нужно сделать**

1. Создать пустую VPC. Выбрать зону.
2. Публичная подсеть.

 - Создать в VPC subnet с названием public, сетью 192.168.10.0/24.
 - Создать в этой подсети NAT-инстанс, присвоив ему адрес 192.168.10.254. В качестве image_id использовать fd80mrhj8fl2oe87o4e1.
 - Создать в этой публичной подсети виртуалку с публичным IP, подключиться к ней и убедиться, что есть доступ к интернету.
3. Приватная подсеть.
 - Создать в VPC subnet с названием private, сетью 192.168.20.0/24.
 - Создать route table. Добавить статический маршрут, направляющий весь исходящий трафик private сети в NAT-инстанс.
 - Создать в этой приватной подсети виртуалку с внутренним IP, подключиться к ней через виртуалку, созданную ранее, и убедиться, что есть доступ к интернету.

### Выполнение задания 1. Yandex Cloud

1. Создаю пустую VPC с именем dvl:

```
resource "yandex_vpc_network" "dvl" {
  name = var.vpc_name

variable "vpc_name" {
  type        = string
  default     = "dvl"
  description = "VPC network"
}
```

2. Создаю в VPC публичную подсеть с названием public, сетью 192.168.10.0/24:

```
resource "yandex_vpc_subnet" "public" {
  name           = var.public_subnet
  zone           = var.default_zone
  network_id     = yandex_vpc_network.dvl.id
  v4_cidr_blocks = var.public_cidr
}

variable "public_cidr" {
  type        = list(string)
  default     = ["192.168.10.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}

variable "public_subnet" {
  type        = string
  default     = "public"
  description = "subnet name"
}
```

Также ресурс и переменные можно посмотреть в файлах [network.tf](https://github.com/RikLedger/13-Project-Cloud-hw-1.md/blob/main/terraform%20/network.tf) и [variables.tf](https://github.com/RikLedger/13-Project-Cloud-hw-1.md/blob/main/terraform%20/variables.tf).

* Создаю в публичной подсети NAT-инстанс, присвоив ему адрес 192.168.10.254. В качестве image_id использую fd80mrhj8fl2oe87o4e1.

Листинг инстанса можно посмотреть в файле [nat_instance.tf](https://github.com/RikLedger/13-Project-Cloud-hw-1.md/blob/main/terraform%20/nat_instance.tf)

* Создаю в публичной подсети виртуальную машину с публичным IP.

Листинг виртуальной машины можно посмотреть в файле [public.tf](https://github.com/RikLedger/13-Project-Cloud-hw-1.md/blob/main/terraform%20/public.tf)

Подключусь к виртуальной машине и проверю, есть ли из неё доступ к интернету:

![img_1](IMG/img_1.png)

![img_2](IMG/img_2.png)

Хост google.com пингуется, интернет на публичной виртуальной машине есть, сеть работает.

3. Создаю в VPC приватную подсеть с названием private, сетью 192.168.20.0/24:

```
resource "yandex_vpc_subnet" "private" {
  name           = var.private_subnet
  zone           = var.default_zone
  network_id     = yandex_vpc_network.dvl.id
  v4_cidr_blocks = var.private_cidr
  route_table_id = yandex_vpc_route_table.private-route.id
}

variable "private_cidr" {
  type        = list(string)
  default     = ["192.168.20.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}

variable "private_subnet" {
  type        = string
  default     = "private"
  description = "subnet name"
}
```
Также ресурс и переменные можно посмотреть в файлах [network.tf](https://github.com/RikLedger/13-Project-Cloud-hw-1.md/blob/main/terraform%20/network.tf) и [variables.tf](https://github.com/RikLedger/13-Project-Cloud-hw-1.md/blob/main/terraform%20/variables.tf).

* Создаю route table. Добавляю статический маршрут, направляющий весь исходящий трафик private сети в NAT-инстанс:

```
resource "yandex_vpc_route_table" "private-route" {
  name       = "private-route"
  network_id = yandex_vpc_network.dvl.id
  static_route {
    destination_prefix = "0.0.0.0/0"
    next_hop_address   = "192.168.10.254"
  }
}
```

* Создаю в приватной подсети виртуальную машину с внутренним IP, внешний IP будет отсутствовать.

Листинг виртуальной машины можно посмотреть в файле [private.tf](https://github.com/RikLedger/13-Project-Cloud-hw-1.md/blob/main/terraform%20/private.tf)

Для проверки доступности интернета на приватной виртуальной машине и работы NAT-инстанса скопирую свой приватный ssh ключ на публичную виртуальную машину. Далее с публичной виртуальной машины подключусь к приватной по внутреннему IP адресу:

![img_3](IMG/img_3.png)

![img_4](IMG/img_4.png)

![img_5](IMG/img_5.png)

Хост google.com пингуется, интернет на приватной виртуальной машине есть, сеть работает.

Выключу виртуальную машину с NAT-инстансом:

![img_6](IMG/img_6.png)

Проверю работу интернета на приватной виртуальной машине:

![img_7](IMG/img_7.png)

Интернет на приватной виртуальной машине перестал работать после отключения NAT-инстанса, следовательно статический маршрут, направляющий весь исходящий трафик private сети в NAT-инстанс был настроен корректно.

После разворачивания инфраструктуры получаем следующие виртуальные машины:

![img_8](IMG/img_8.png)

Output вывод Terraform выглядит следующим образом:

![img_9](IMG/img_9.png)
