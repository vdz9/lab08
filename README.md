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
