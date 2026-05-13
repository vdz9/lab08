[![CI](https://github.com/vdz9/lab04/actions/workflows/ci.yml/badge.svg)](https://github.com/vdz9/lab04/actions/workflows/ci.yml)
### Tasks: Настройка автоматической сборки проекта через Github Actions

1. Создать публичный репозиторий с названием **lab04** на сервисе **GitHub**
2. Ознакомиться со ссылками учебного материала
3. Включить интеграцию сервиса **Github Actions** с созданным репозиторием
4. Обновить токен, добавив права **workflow**
5. Выполнить инструкцию учебного материала

### 1. Настройка переменных окружения

```bash
export GITHUB_USERNAME=vdz9
export GITHUB_TOKEN=<сохраненный_токен>
source scripts/activate

cd ${GITHUB_USERNAME}/workspace
pushd .
source scripts/activate

git clone https://github.com/${GITHUB_USERNAME}/lab03.git projects/lab04
cd projects/lab04
git remote remove origin
git remote add origin https://github.com/${GITHUB_USERNAME}/lab04.git
```

Результат: Копирование репозитория из предыдущей лабораторной работы в текущую и последующая его привязка к новому репозиторию
### 2. Создание директории для рабочих процессов GitHub Actions

```
mkdir -p .github/workflows
```

Результат: Создание директории .github/workflows для хранения конфигураций CI

### 3. Создание конфигурации CI для сборки с GCC

```
cat > .github/workflows/ci.yml <<EOF
name: CI

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      CC: gcc
      CXX: g++
    steps:
    - uses: actions/checkout@v3
    
    - name: Install CMake
      run: sudo apt-get install -y cmake cmake-data
    
    - name: Configure
      run: cmake -H. -B_build -DCMAKE_INSTALL_PREFIX=_install
    
    - name: Build
      run: cmake --build _build
    
    - name: Install
      run: cmake --build _build --target install
    
    - name: Upload artifacts
      uses: actions/upload-artifact@v3
      with:
        name: install-dir
        path: _install
EOF
```
Результат: Создание файла ci.yml с настройкой CI для сборки проекта с компилятором GCC (g++).

### 4. Добавление значка статуса CI

```
ex -sc '1i|[![CI](https://github.com/vdz9/lab04/actions/workflows/ci.yml/badge.svg)](https://github.com/vdz9/lab04/actions/workflows/ci.yml)' -cx README.md
```

Результат: Добавлен значок статуса CI
