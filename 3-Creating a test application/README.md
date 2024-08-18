# Создание тестового приложения
<details>
	<summary></summary>
      <br>

Для перехода к следующему этапу необходимо подготовить тестовое приложение, эмулирующее основное приложение разрабатываемое вашей компанией.

Способ подготовки:

1. Рекомендуемый вариант:  
   а. Создайте отдельный git репозиторий с простым nginx конфигом, который будет отдавать статические данные.  
   б. Подготовьте Dockerfile для создания образа приложения.  
2. Альтернативный вариант:  
   а. Используйте любой другой код, главное, чтобы был самостоятельно создан Dockerfile.

Ожидаемый результат:

1. Git репозиторий с тестовым приложением и Dockerfile.
2. Регистри с собранным docker image. В качестве регистри может быть DockerHub или [Yandex Container Registry](https://cloud.yandex.ru/services/container-registry), созданный также с помощью terraform.

</details>

---
## Решение:

Подготовим тестовое приложение.

Созададим отдельный git репозиторий `Test-application` с простым nginx конфигом, который будет отдавать статические данные:

Клонируем репозиторий:
```bash
git clone https://github.com/BaryshnikovNV/Test-application.git
```

Создадим в этом репозитории файл содержащую HTML-код ниже:  
index.html
```html
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
```

Создадим Dockerfile, который будет запускать веб-сервер Nginx в фоне с индекс страницей:  
Dockerfile
```Dockerfile
FROM nginx:1.27-alpine

COPY index.html /usr/share/nginx/html
```

Загрузим файлы в [Git-репозиторий](https://github.com/BaryshnikovNV/Test-application).

Создадим папку для приложения `mkdir mynginx` и скопируем в нее ранее созданые файлы.  
В этой папке выполним сборку приложения:
<details>
	<summary></summary>
      <br>

```bash
baryshnikov@compute-vm-2-4-20-hdd-1723564389639:~/mynginx$ sudo docker build -t baryshnikov/nginx:v1 .
[+] Building 6.4s (7/7) FINISHED                                                                                                             docker:default
 => [internal] load build definition from Dockerfile                                                                                                   0.1s
 => => transferring dockerfile: 99B                                                                                                                    0.0s
 => [internal] load metadata for docker.io/library/nginx:1.27-alpine                                                                                   1.8s
 => [internal] load .dockerignore                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                        0.0s
 => [internal] load build context                                                                                                                      0.0s
 => => transferring context: 128B                                                                                                                      0.0s
 => [1/2] FROM docker.io/library/nginx:1.27-alpine@sha256:208b70eefac13ee9be00e486f79c695b15cef861c680527171a27d253d834be9                             2.3s
 => => resolve docker.io/library/nginx:1.27-alpine@sha256:208b70eefac13ee9be00e486f79c695b15cef861c680527171a27d253d834be9                             0.0s
 => => sha256:208b70eefac13ee9be00e486f79c695b15cef861c680527171a27d253d834be9 9.07kB / 9.07kB                                                         0.0s
 => => sha256:1ae23480369fa4139f6dec668d7a5a941b56ea174e9cf75e09771988fe621c95 11.01kB / 11.01kB                                                       0.0s
 => => sha256:b3ee43e51ca6c92d08e40eaf4d7dbc50444ba57c1a200c437a3c86e5a7631ba6 627B / 627B                                                             0.5s
 => => sha256:a377278b7dde3a8012b25d141d025a88dbf9f5ed13c5cdf21ee241e7ec07ab57 2.50kB / 2.50kB                                                         0.0s
 => => sha256:46b060cc26202cf98e28414d790b5cabd67094bba50315a1ae2e9daf913fca4f 3.42MB / 3.42MB                                                         0.3s
 => => sha256:21af147d2ad5905372d6284d8ac5031599e2983fa2cc9f8251f9130fdf6bce19 1.92MB / 1.92MB                                                         0.4s
 => => extracting sha256:46b060cc26202cf98e28414d790b5cabd67094bba50315a1ae2e9daf913fca4f                                                              0.2s
 => => sha256:b17a9d410da1886001947f5f826043292df395277cb23e17970f5b82a6b486ed 956B / 956B                                                             0.5s
 => => sha256:542e3e75411d1223efe21092951a0b87b85cb4377accf8f360d635d23dea72a9 393B / 393B                                                             0.6s
 => => sha256:a5e22afba545a92d46609059fe9fe2b90028b9f3fb7c78be28cb6d4ed9e53fd3 1.40kB / 1.40kB                                                         0.7s
 => => sha256:2b2faad386dfd2da1e19aa6e0d91d428b849181de439c0b289f383816c812304 1.21kB / 1.21kB                                                         0.7s
 => => extracting sha256:21af147d2ad5905372d6284d8ac5031599e2983fa2cc9f8251f9130fdf6bce19                                                              0.2s
 => => sha256:fb923a41dc10df4d74119907e9112426c8e0e2ce3d6851c4e2dcfb7e0765861b 13.04MB / 13.04MB                                                       1.1s
 => => extracting sha256:b3ee43e51ca6c92d08e40eaf4d7dbc50444ba57c1a200c437a3c86e5a7631ba6                                                              0.0s
 => => extracting sha256:b17a9d410da1886001947f5f826043292df395277cb23e17970f5b82a6b486ed                                                              0.0s
 => => extracting sha256:542e3e75411d1223efe21092951a0b87b85cb4377accf8f360d635d23dea72a9                                                              0.0s
 => => extracting sha256:2b2faad386dfd2da1e19aa6e0d91d428b849181de439c0b289f383816c812304                                                              0.0s
 => => extracting sha256:a5e22afba545a92d46609059fe9fe2b90028b9f3fb7c78be28cb6d4ed9e53fd3                                                              0.0s
 => => extracting sha256:fb923a41dc10df4d74119907e9112426c8e0e2ce3d6851c4e2dcfb7e0765861b                                                              0.4s
 => [2/2] COPY index.html /usr/share/nginx/html                                                                                                        1.6s
 => exporting to image                                                                                                                                 0.1s
 => => exporting layers                                                                                                                                0.1s
 => => writing image sha256:93d209aa730a03204d80d3feb9a38d4cc20545d406115d5dac9dd9ca426ec7f0                                                           0.0s
 => => naming to docker.io/baryshnikov/nginx:v1
```

</details>

Проверим, что образ создался:
```bash
baryshnikov@compute-vm-2-4-20-hdd-1723564389639:~/mynginx$ sudo docker images
REPOSITORY          TAG       IMAGE ID       CREATED         SIZE
baryshnikov/nginx   v1        93d209aa730a   5 minutes ago   43.2MB
```

Запустим docker-контейнер с созданным образом и проверим его  работоспособность:
```bash
baryshnikov@compute-vm-2-4-20-hdd-1723564389639:~/mynginx$ sudo docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                               NAMES
7e7d371b7d27   baryshnikov/nginx:v1   "/docker-entrypoint.…"   15 seconds ago   Up 15 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   app
baryshnikov@compute-vm-2-4-20-hdd-1723564389639:~/mynginx$
baryshnikov@compute-vm-2-4-20-hdd-1723564389639:~/mynginx$
baryshnikov@compute-vm-2-4-20-hdd-1723564389639:~/mynginx$ curl http://89.169.147.229
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
```

Загрузим созданный образ в реестре [Docker Hub](https://hub.docker.com/layers/baryshnikovnv/nginx/v1/images/sha256-3058e9f2b17c25ab5f7b6ec42577c3db658890a8f3673b5a9ae092a9aed73fcd?context=repo):
```bash
baryshnikov@compute-vm-2-4-20-hdd-1723564389639:~/mynginx$ sudo docker push baryshnikovnv/nginx:v1
The push refers to repository [docker.io/baryshnikovnv/nginx]
16f5cd97d8ef: Pushed
28d40eb13793: Pushed
2ee64cbdc81d: Pushed
a0bde08c3815: Pushed
3be2be874bba: Pushed
fb5df5db7bbd: Pushed
eadc278e8f9e: Pushed
9dca7439e1b3: Pushed
b895814e9e64: Pushed
v1: digest: sha256:3058e9f2b17c25ab5f7b6ec42577c3db658890a8f3673b5a9ae092a9aed73fcd size: 2196
```