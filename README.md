[![CI](https://github.com/vdz9/lab06/actions/workflows/ci.yml/badge.svg)](https://github.com/vdz9/lab06/actions/workflows/ci.yml)
### Tasks: Изучение систем автоматизации развёртывания и управления приложениями на примере Docker

1. Создать публичный репозиторий с названием **lab08** на сервисе **GitHub**
2. Ознакомиться со ссылками учебного материала
3. Выполнить инструкцию учебного материала

### 1. Настройка переменных окружения

```bash
export GITHUB_USERNAME=vdz9
export GITHUB_TOKEN=<сохраненный_токен>
source scripts/activate

cd ${GITHUB_USERNAME}/workspace
pushd .
source scripts/activate

git clone https://github.com/${GITHUB_USERNAME}/lab07.git projects/lab08
cd projects/lab08
git remote remove origin
git remote add origin https://github.com/${GITHUB_USERNAME}/lab08.git
```

Результат: Копирование репозитория из предыдущей лабораторной работы в текущую и последующая его привязка к новому репозиторию
### 2. Создание Dockerfile

```
cat > Dockerfile <<'EOF'
FROM ubuntu:20.04
EOF
```
```
cat >> Dockerfile <<'EOF'
RUN apt update && apt install -y gcc g++ cmake
EOF
```
```
cat >> Dockerfile <<'EOF'
COPY . print/
WORKDIR print
EOF
```
```
cat >> Dockerfile <<'EOF'
RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install
EOF
```
```
cat >> Dockerfile <<'EOF'
ENV LOG_PATH /home/logs/log.txt
EOF
```
```
cat >> Dockerfile <<'EOF'
VOLUME /home/logs
EOF
```
```
cat >> Dockerfile <<'EOF'
WORKDIR _install/bin
EOF
```
```
cat >> Dockerfile <<'EOF'
ENTRYPOINT ["./demo"]
EOF
```

Результат: Создание Dockerfile с инструкциями для сборки образа на основе Ubuntu 20.04, установки компиляторов, копирования исходного кода, сборки проекта и настройки точки входа на демо-приложение

### 3. Сборка Docker-образа

```
sudo docker build -t logger .
```
Результат: Успешная сборка Docker-образа с именем logger

### 4. Проверка списка образов

```
sudo docker images
```
Результат: 
```
IMAGE           ID             DISK USAGE   CONTENT SIZE   EXTRA
logger:latest   9a51c735b0de        652MB          182MB        
ubuntu:20.04    8feb4d8ca535        111MB         29.3MB
```

### 5. Запуск контейнера с томом для логов

```
mkdir -p logs
sudo docker run -it -v "$(pwd)/logs/:/home/logs/" logger
```

Результат: Запуск контейнера в интерактивном режиме, введенный текст сохранен в файл /home/logs/log.txt внутри контейнера, который смонтирован на локальную директорию logs  Ввод текста:
```
text1
text2
text3
```
### 6. Проверка информации об образе
```
sudo docker inspect logger
```
Результат:
```
[
    {
        "Architecture": "amd64",
        "Config": {
            "Labels": {
                "org.opencontainers.image.ref.name": "ubuntu",
                "org.opencontainers.image.version": "20.04"
        },
        "Created": "2026-05-14T22:46:23.009217196+03:00",
        "Descriptor": {
            "digest": "sha256:9a51c735b0dee24f0d55633c45caf5adda94a0922c524dd9b20d10f8897f3674",
            "mediaType": "application/vnd.oci.image.manifest.v1+json",
            "size": 1624
        },
        "Id": "sha256:9a51c735b0dee24f0d55633c45caf5adda94a0922c524dd9b20d10f8897f3674",
        "Metadata": {
            "LastTagTime": "2026-05-14T19:46:23.090314389Z"
        },
        "Os": "linux",
        "Parent": "sha256:2f6564a42107007e524407904ba950d5e92463e802b6a7468b972898ebf307db",
        "RepoDigests": [
            "logger@sha256:9a51c735b0dee24f0d55633c45caf5adda94a0922c524dd9b20d10f8897f3674"
        ],
        "RepoTags": [
            "logger:latest"
        ],
        "RootFS": {
            "Layers": [
                "sha256:470b66ea5123c93b0d5606e4213bf9e47d3d426b640d32472e4ac213186c4bb6",
                "sha256:16b8634b3731e839ddfc56e54218071256344f0d6d0ad97f43c84812995e6f34",
                "sha256:026c51be27ef5bd06cff92fc321066965f6ed40f33c8d2247e030454760910f0",
                "sha256:08b18aeb56a2f19f3c248f86af46680ed9c057f7f33436b42442a60f8b8eda57",
                "sha256:3125a2c7778d1e6d94180474c3670844a86d8dbe2994964ee24f21e01c74cde6",
                "sha256:24edf523f5171f0d3544ae19dce718346a79528fea785db85e0bc7a6b9ae4c1c"
            ],
            "Type": "layers"
        },
        "Size": 182479038
    }
]
#docker inspect немного отредактирован чтобы не раскрыть внутреннюю структуру проекта, хэш образы, ключи API и т.д.
```
### 7. Проверка сохраненных логов

```
cat logs/log.txt
```
Результат: 
```
text1
text2
text3
```
