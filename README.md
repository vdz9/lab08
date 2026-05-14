[![CI](https://github.com/vdz9/lab06/actions/workflows/ci.yml/badge.svg)](https://github.com/vdz9/lab06/actions/workflows/ci.yml)
### Tasks: Изучение систем управления пакетами на примере Hunter

1. Создать публичный репозиторий с названием **lab06** на сервисе **GitHub**
2. Ознакомиться со ссылками учебного материала
3. Выполнить инструкцию учебного материала

```
В лабораторной работе используется пакетный менеджер Hunter для загрузки GTest, из-за проблем с совместимостью в данной работе Hunter заменен на прямую интеграцию GTest через git submodule
```

### 1. Настройка переменных окружения

```bash
export GITHUB_USERNAME=vdz9
export GITHUB_TOKEN=<сохраненный_токен>
source scripts/activate

cd ${GITHUB_USERNAME}/workspace
pushd .
source scripts/activate

git clone https://github.com/${GITHUB_USERNAME}/lab06.git projects/lab07
cd projects/lab07
git remote remove origin
git remote add origin https://github.com/${GITHUB_USERNAME}/lab07.git
```

Результат: Копирование репозитория из предыдущей лабораторной работы в текущую и последующая его привязка к новому репозиторию
### 2. Добавление GTest как подмодуля

```
mkdir -p third-party
git submodule add https://github.com/google/googletest third-party/gtest
cd third-party/gtest
git fetch --tags
git checkout tags/v1.12.1 -b release-1.12.1
cd ../..
git add third-party/gtest .gitmodules
git commit -m "added gtest v1.12.1 as submodule"
```

### 3. Настройка CMakeLists.txt для поддержки тестов

```
sed -i '/option(BUILD_EXAMPLES "Build examples" OFF)/a\option(BUILD_TESTS "Build tests" OFF)' CMakeLists.txt
```
```
cat >> CMakeLists.txt <<'EOF'
if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(third-party/gtest)
  file(GLOB ${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
  add_executable(check ${${PROJECT_NAME}_TEST_SOURCES})
  target_link_libraries(check ${PROJECT_NAME} gtest_main)
  add_test(NAME check COMMAND check)
endif()
EOF
```
Результат: Добавление опция BUILDTESTS и блок сборки тестов с прямым подключением GTest через add_subdirectory

### 4. Создание модульного теста

```
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

### 5. Сборка и запуск тестов

```
cmake -H. -B_build -DBUILD_TESTS=ON
cmake --build _build
cmake --build _build --target check
cmake --build _build --target test
cmake --build _build --target test -- ARGS=--verbose
```

Результат: Проект сконфигурирован с включенными тестами, cборка выполнена успешно

### 6. Результат теста
```
Start 1: check

1: Test command: /home/vdz/vdz9/workspace/projects/lab07/_build/check
1: Working Directory: /home/vdz/vdz9/workspace/projects/lab07/_build
1: Test timeout computed to be: 10000000
1: Running main() from /home/vdz/vdz9/workspace/projects/lab07/third-party/gtest/googletest/src/gtest_main.cc
1: [==========] Running 1 test from 1 test suite.
1: [----------] Global test environment set-up.
1: [----------] 1 test from Print
1: [ RUN      ] Print.InFileStream
1: [       OK ] Print.InFileStream (0 ms)
1: [----------] 1 test from Print (0 ms total)
1: 
1: [----------] Global test environment tear-down
1: [==========] 1 test from 1 test suite ran. (0 ms total)
1: [  PASSED  ] 1 test.
1/1 Test #1: check ............................   Passed    0.01 sec
```

### 7. Cоздание демонстрационного приложения

```
mkdir demo
cat > demo/main.cpp <<EOF
#include <print.hpp>

#include <cstdlib>

int main(int argc, char* argv[])
{
  const char* log_path = std::getenv("LOG_PATH");
  if (log_path == nullptr)
  {
    std::cerr << "undefined environment variable: LOG_PATH" << std::endl;
    return 1;
  }
  std::string text;
  while (std::cin >> text)
  {
    std::ofstream out{log_path, std::ios_base::app};
    print(text, out);
    out << std::endl;
  }
}
EOF
```
```
sed -i '/add_executable(example2/a\
\
add_executable(demo ${CMAKE_CURRENT_SOURCE_DIR}/demo/main.cpp)\
target_link_libraries(demo print)\
install(TARGETS demo RUNTIME DESTINATION bin)' CMakeLists.txt
```

Результат: Создание демонстрационного приложения demo, которое читает текст из стандартного ввода и записывает LOG_PATH

### 8. Сборка демо-приложения

```cmake -H. -B_build
cmake --build _build
echo "test message" | LOG_PATH=/tmp/test.log _build/demo
cat /tmp/test.log
```

Результат: Демо-приложение собрано и протестировано, текст успешно записан в лог-файл
