[![CI](https://github.com/vdz9/lab06/actions/workflows/ci.yml/badge.svg)](https://github.com/vdz9/lab06/actions/workflows/ci.yml)
### Tasks: Изучение средств пакетирования на примере CPack

1. Создать публичный репозиторий с названием **lab06** на сервисе **GitHub**
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

git clone https://github.com/${GITHUB_USERNAME}/lab05.git projects/lab06
cd projects/lab06
git remote remove origin
git remote add origin https://github.com/${GITHUB_USERNAME}/lab06.git
```

Результат: Копирование репозитория из предыдущей лабораторной работы в текущую и последующая его привязка к новому репозиторию
### 2. Настройка версионирования в CMakeLists.txt

```
gsed -i '/project(print)/a\
set(PRINT_VERSION_MAJOR 0)
' CMakeLists.txt
gsed -i '/project(print)/a\
set(PRINT_VERSION_MINOR 1)
' CMakeLists.txt
gsed -i '/project(print)/a\
set(PRINT_VERSION_PATCH 0)
' CMakeLists.txt
gsed -i '/project(print)/a\
set(PRINT_VERSION_TWEAK 0)
' CMakeLists.txt
gsed -i '/project(print)/a\
set(PRINT_VERSION\
  \${PRINT_VERSION_MAJOR}.\${PRINT_VERSION_MINOR}.\${PRINT_VERSION_PATCH}.\${PRINT_VERSION_TWEAK})
' CMakeLists.txt
gsed -i '/project(print)/a\
set(PRINT_VERSION_STRING "v\${PRINT_VERSION}")
' CMakeLists.txt
git diff
```

Результат: Добавление переменных  версий в CMakeLists.txt: MAJOR=0, MINOR=1, PATCH=0, TWEAK=0. Итоговая версия - 0.1.0.0

### 3. Создание DESCRIPTION

```
cat > DESCRIPTION <<EOF
Static C++ library for printing text to console and files.
Provides print() function with ostream and ofstream support.
EOF
```
Результат: Создание файла DESCRIPTION 

### 4. Создание ChangeLog.md

```
touch ChangeLog.md
export DATE="`LANG=en_US date +'%a %b %d %Y'`"
cat > ChangeLog.md <<EOF
* ${DATE} ${GITHUB_USERNAME} <${GITHUB_EMAIL}> 0.1.0.0
- Initial RPM release
EOF
```

Результат: Создание файла ChangeLog.md

### 5. Создание CPack
```
cat > CPackConfig.cmake <<EOF
include(InstallRequiredSystemLibraries)
EOF
```
```
cat >> CPackConfig.cmake <<EOF
set(CPACK_PACKAGE_CONTACT ${GITHUB_EMAIL})
set(CPACK_PACKAGE_VERSION_MAJOR \${PRINT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR \${PRINT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH \${PRINT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_TWEAK \${PRINT_VERSION_TWEAK})
set(CPACK_PACKAGE_VERSION \${PRINT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_FILE \${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for printing")
EOF
```
```
cat >> CPackConfig.cmake <<EOF
set(CPACK_RESOURCE_FILE_LICENSE \${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_RESOURCE_FILE_README \${CMAKE_CURRENT_SOURCE_DIR}/README.md)
EOF
```
```
cat >> CPackConfig.cmake <<EOF
set(CPACK_RPM_PACKAGE_NAME "print-devel")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "print")
set(CPACK_RPM_CHANGELOG_FILE \${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)
set(CPACK_RPM_PACKAGE_RELEASE 1)
EOF
```
```
cat >> CPackConfig.cmake <<EOF
set(CPACK_DEBIAN_PACKAGE_NAME "libprint-dev")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)
EOF
```
```
cat >> CPackConfig.cmake <<EOF
include(CPack)
EOF
```
```
cat >> CMakeLists.txt <<EOF
include(CPackConfig.cmake)
EOF
```

Результат: Создание конфигурации CPack, файла CPackConfig.cmake с настройками для генерации DEB и RPM пакетов

### 6. Локальная сборка пакета TGZ и cохранение артефактов
```
cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
cmake --build _build --target package
```
```
mkdir -p artifacts
mv _build/*.tar.gz artifacts
tree artifacts
```

Результат: Конфигурация проекта с генератором TGZ. Архив перемещен в директорию artifacts.

### 7. Итоговая структура

```
artifacts/
└── print-0.1.0.0-Linux.tar.gz

1 directory, 1 file
```

### =====HOMEWORK=====

### 1. Создание библиотеки formatter

```
mkdir -p formatter_lib
cat > formatter_lib/formatter.cpp <<'EOF'
#include "formatter.h"
std::string formatter(const std::string& text) { return text; }
EOF
```
```
cat > formatter_lib/formatter.h <<'EOF'
#pragma once
#include <string>
std::string formatter(const std::string& text);
EOF
```
```
cat > formatter_lib/CMakeLists.txt <<'EOF'
cmake_minimum_required(VERSION 3.5)
project(formatter)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_library(formatter STATIC formatter.cpp)
EOF
```

### 2. Создание библиотеки formatter_ex

```
mkdir -p formatter_ex_lib
cat > formatter_ex_lib/formatter_ex.cpp <<'EOF'
#include "formatter_ex.h"
std::string formatter_ex(const std::string& text) { return formatter(text); }
EOF
```
```
cat > formatter_ex_lib/formatter_ex.h <<'EOF'
#pragma once
#include <string>
#include "formatter.h"
std::string formatter_ex(const std::string& text);
EOF
```
```
cat > formatter_ex_lib/CMakeLists.txt <<'EOF'
cmake_minimum_required(VERSION 3.5)
project(formatter_ex)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_library(formatter_ex STATIC formatter_ex.cpp)
include_directories(${CMAKE_SOURCE_DIR}/formatter_lib)
target_link_libraries(formatter_ex formatter)
EOF
```

### 3. Создание библиотеки solver_lib

```
mkdir -p solver_lib
cat > solver_lib/solver.cpp <<'EOF'
#include "solver.h"
double solver(double a, double b) { return a + b; }
EOF
```
```
cat > solver_lib/solver.h <<'EOF'
#pragma once
double solver(double a, double b);
EOF
```
```
cat > solver_lib/CMakeLists.txt <<'EOF'
cmake_minimum_required(VERSION 3.5)
project(solver_lib)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_library(solver_lib STATIC solver.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
EOF
```

### 4. Обновление CMakeLists.txt для solver_application

```
cat > solver_application/CMakeLists.txt <<'EOF'
cmake_minimum_required(VERSION 3.5)
project(solver)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_executable(solver equation.cpp)
include_directories(${CMAKE_SOURCE_DIR}/formatter_lib)
include_directories(${CMAKE_SOURCE_DIR}/formatter_ex_lib)
include_directories(${CMAKE_SOURCE_DIR}/solver_lib)
target_link_libraries(solver formatter_ex solver_lib formatter)
EOF
```

Результат: Обновленение CMakeLists.txt для приложения solver с правильными путями к заголовочным файлам и линковкой всех библиотек

### 5. Настройка упаковки solver в CPack

```
cat >> CPackConfig.cmake <<'EOF'
set(CPACK_PACKAGE_NAME "solver")
set(CPACK_RPM_PACKAGE_NAME "solver")
set(CPACK_DEBIAN_PACKAGE_NAME "solver")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Solver application with formatter")
set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "solver-\${CPACK_PACKAGE_VERSION}-Source")
EOF
```

Результат:Результат: Обновление CPackConfig.cmake для создания пакетов solver в форматах DEB, RPM, TGZ, ZIP


### 6. Настройка CI для автоматических релизов

```
mkdir -p .github/workflows
cat > .github/workflows/solver-release.yml <<'EOF'
name: Solver Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    name: Create Solver Release
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Create Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Solver ${{ github.ref }}
          draft: false
          prerelease: false

  build-packages:
    name: Build Solver Packages
    needs: release
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        include:
          - os: ubuntu-latest
            generator: DEB;RPM;TGZ
          - os: macos-latest
            generator: DragNDrop;TGZ
          - os: windows-latest
            generator: WIX;NSIS;ZIP
    
    steps:
      - uses: actions/checkout@v4

      - name: Install dependencies (Linux RPM)
        if: runner.os == 'Linux'
        run: sudo apt-get install -y rpm

      - name: Configure
        run: cmake -H. -B_build -DCPACK_GENERATOR="${{ matrix.generator }}"

      - name: Build
        run: cmake --build _build

      - name: Package
        run: cmake --build _build --target package

      - name: Upload Release Asset
        uses: softprops/action-gh-release@v1
        with:
          files: _build/*.tar.gz;_build/*.deb;_build/*.rpm;_build/*.dmg;_build/*.msi;_build/*.zip
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
EOF
```

Результат: Создание рабочего процесса GitHub Actions для автоматической сборки пакетов solver

### 7. Локальная сборка

```rm -rf _build
cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
cmake --build _build
cmake --build _build --target package
```

Результат: Проект успешно собран.
