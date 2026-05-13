### Tasks: Bзучение систем автоматизации сборки проекта на примере **CMake**

1. Создать публичный репозиторий с названием **lab03** на сервисе **GitHub**
2. Ознакомиться со ссылками учебного материала
3. Выполнить инструкцию учебного материала

### 1. Настройка переменных окружения

```bash
export GITHUB_USERNAME=vdz9
export GITHUB_EMAIL=<адрес_почтового_ящика>
export GITHUB_TOKEN=<сохраненный_токен>
export CMAKE_CURRENT_SOURCED_DIR=<директория_с_файлом_CMake>
source scripts/activate

cd ${GITHUB_USERNAME}/workspace
pushd .
source scripts/activate

git clone https://github.com/${GITHUB_USERNAME}/lab02.git projects/lab03
cd projects/lab03
git remote remove origin
git remote add origin https://github.com/${GITHUB_USERNAME}/lab03.git
```

Результат: Копирование репозитория из предыдущей лабораторной работы в текущую и последующая его привязка к новому репозиторию
### 2. Ручная компиляция

```
g++ -std=c++11 -I./include -c sources/print.cpp
ls print.o
nm print.o | grep print
ar rvs print.a print.o
file print.a
g++ -std=c++11 -I./include -c examples/example1.cpp
ls example1.o
g++ example1.o print.a -o example1
./example1 && echo
```
```
g++ -std=c++11 -I./include -c examples/example2.cpp
nm example2.o
g++ example2.o print.a -o example2
./example2
cat log.txt && echo
```
```
rm -rf example1.o example2.o print.o
rm -rf print.a
rm -rf example1 example2
rm -rf log.txt
```

Результат: Компиляция файлов из предыдущей лабораторной работы, создание объектного файла print.o и статической библиотеки print.a, запуск приложений example1 и example2, программа example2 создала файл log.txt с выводом "hello", удаление всех временных файлов компиляции

### 3. Создание CMakeLists.txt для статической библиотеки

```
cat > CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.4)
project(print)
EOF
```
```
cat >> CMakeLists.txt <<EOF
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
EOF
```
```
cat >> CMakeLists.txt <<EOF
add_library(print STATIC ${CMAKE_CURRENT_SOURCE_DIR}/sources/print.cpp)
EOF

```
```
cat >> CMakeLists.txt <<EOF
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
EOF
```
```
cmake -H. -B_build
cmake --build _build
```
Результат: Создан файл CMakeLists.txt

### 4. Добавление целей для исполняемых файлов

```
cat >> CMakeLists.txt <<EOF
add_executable(example1 ${CMAKE_CURRENT_SOURCE_DIR}/examples/example1.cpp)
add_executable(example2 ${CMAKE_CURRENT_SOURCE_DIR}/examples/example2.cpp)
EOF
```
```
cat >> CMakeLists.txt <<EOF
target_link_libraries(example1 print)
target_link_libraries(example2 print)
EOF
```
```
cmake --build _build
cmake --build _build --target print
cmake --build _build --target example1
cmake --build _build --target example2
```

Результат: В CMakeLists.txt добавлены цели example1 и example2 с привязкой к библиотеке print, сборка всех файлов выполнена успешно

### 5. Тестирование собранных приложений

```
ls -la _build/libprint.a
_build/example1 && echo
_build/example2
cat log.txt && echo
rm -rf log.txt
```

Результат: Статическая библиотека libprint.a создана в _build, приложение example1 вывело "hello", example2 создало log.txt с "hello", после проверки log.txt удален

### 6. Установка CMakeLists.txt

```
git clone https://github.com/tp-labs/lab03 tmp
mv -f tmp/CMakeLists.txt .
rm -rf tmp

cat CMakeLists.txt
cmake -H. -B_build -DCMAKE_INSTALL_PREFIX=_install
cmake --build _build --target install
tree _install
```

Результат: CMakeLists.txt загружен и заменен. Выполнена установка проекта в _install

### 7. Итоговая структура

```
_install
├── cmake
│   ├── print-config.cmake
│   └── print-config-noconfig.cmake
├── include
│   └── print.hpp
└── lib
    └── libprint.a

4 directories, 4 files
```
