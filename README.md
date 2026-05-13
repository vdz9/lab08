[![CI](https://github.com/vdz9/lab05/actions/workflows/ci.yml/badge.svg)](https://github.com/vdz9/lab05/actions/workflows/ci.yml)
### Tasks: Изучение фреймворков для тестирования на примере GTest

1. Создать публичный репозиторий с названием **lab05** на сервисе **GitHub**
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

git clone https://github.com/${GITHUB_USERNAME}/lab04.git projects/lab05
cd projects/lab05
git remote remove origin
git remote add origin https://github.com/${GITHUB_USERNAME}/lab05.git
```

Результат: Копирование репозитория из предыдущей лабораторной работы в текущую и последующая его привязка к новому репозиторию
### 2. Добавление фреймворка GTest

```
mkdir -p third-party
git submodule add https://github.com/google/googletest third-party/gtest
cd third-party/gtest && git checkout release-1.12.1 && cd ../..
git add third-party/gtest
git commit -m "added gtest framework"
```

Результат: Добавление Google Test как подмодуля в директорию third-party/gtest

### 3. Настройка CMakeLists.txt для поддержки тестов

```
sed -i '/option(BUILD_EXAMPLES "Build examples" OFF)/a\option(BUILD_TESTS "Build tests" OFF)' CMakeLists.txt
```
```
cat >> CMakeLists.txt <<EOF
if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(third-party/gtest)
  file(GLOB \${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
  add_executable(check \${\${PROJECT_NAME}_TEST_SOURCES})
  target_link_libraries(check \${PROJECT_NAME} gtest_main)
  add_test(NAME check COMMAND check)
endif()
EOF
```
Результат: Добавление в CMakeLists.txt опции BUILD_TESTS и блока сборки тестов с использованием GTest

### 4. Создание модульного теста

```bash
mkdir tests
cat > tests/test1.cpp <<EOF
#include <print.hpp>
#include <gtest/gtest.h>

TEST(Print, InFileStream)
{
  std::string filepath = "file.txt";
  std::string text = "hello";
  std::ofstream out{filepath};
  print(text, out);
  out.close();
  std::string result;
  std::ifstream in{filepath};
  in >> result;
  EXPECT_EQ(result, text);
}
EOF
```

Результат: Создание файла test1.cpp с тестом для функции print

### 5. Запуск тестов
```
cmake -H. -B_build -DBUILD_TESTS=ON
cmake --build _build
cmake --build _build --target check
cmake --build _build --target test
cmake --build _build --target test -- ARGS=--verbose
```

Результат: Успешный запуск тестов

### 6. Обновление CI конфигурации GitHub Actions

```
cat > .github/workflows/ci.yml <<'EOF'
---
name: CI

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      CC: gcc
      CXX: g++
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Install CMake
        run: sudo apt-get install -y cmake cmake-data

      - name: Configure
        run: cmake -H. -B_build -DCMAKE_INSTALL_PREFIX=_install -DBUILD_TESTS=ON

      - name: Build
        run: cmake --build _build

      - name: Install
        run: cmake --build _build --target install

      - name: Test
        run: cmake --build _build --target test -- ARGS=--verbose

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: install-dir
          path: _install
EOF
```

Результат: Добавление флага BUILD_TESTS=ON и шага запуска тестов с подробным выводом в файл ci.yml


