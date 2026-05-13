### Tasks: Изучение систем контроля версий на примере **Git**.

1. Создать публичный репозиторий с названием **lab02** и с лиценцией **MIT**
2. Сгенирировать токен для доступа к сервису **GitHub** с правами **repo**
3. Ознакомиться со ссылками учебного материала
4. Выполнить инструкцию учебного материала

### 1. Настройка переменных окружения

```bash
export GITHUB_USERNAME=vdz9
export GITHUB_EMAIL=<адрес_почтового_ящика>
export GITHUB_TOKEN=<сохраненный_токен>
alias edit=nano
```

### 2. Настройка переменных окружения

```
mkdir -p ~/.config
cat > ~/.config/hub <<EOF
github.com:
- user: ${GITHUB_USERNAME}
  oauth_token: ${GITHUB_TOKEN}
  protocol: https
EOF
git config --global hub.protocol https
```

Результат: hub настроен для взаимодействия с GitHub

### 3. Инициализация и настройка локального репозитория

```
cd ${GITHUB_USERNAME}/workspace
source scripts/activate

mkdir -p projects/lab02 && cd projects/lab02
git init

git config --global user.name ${GITHUB_USERNAME}
git config --global user.email ${GITHUB_EMAIL}

git config -e --global
```

Результат: Инициаkbлизация пустой Git-репозитория

### 4. Cинхронизация

```
git remote add origin https://github.com/vzd9/lab02.git

touch README.md
git status
git add README.md
git commit -m "added README.md"
git push -u origin main
```

Результат: Создание и отправка на GitHub первого коммита с пустым файлом README.md

### 5. Работа с .gitignore

```shell
#Через веб-интерфейс был добавлен файл .gitignore
git pull origin main
git log
```

Результат: Появление в истории коммитов записи о добавлении .gitignore

### 6. Создание структуры проекта и исходных файлов

```
mkdir sources include examples
```
```bash
cat > sources/print.cpp <<EOF
#include <print.hpp>

void print(const std::string& text, std::ostream& out)
{
  out << text;
}

void print(const std::string& text, std::ofstream& out)
{
  out << text;
}
EOF
```
```bash
cat > include/print.hpp <<EOF
#include <fstream>
#include <iostream>
#include <string>

void print(const std::string& text, std::ofstream& out);
void print(const std::string& text, std::ostream& out = std::cout);
EOF
```
```bash
cat > examples/example1.cpp <<EOF
#include <print.hpp>

int main(int argc, char** argv)
{
  print("hello");
}
EOF
```
```bash
cat > examples/example2.cpp <<EOF
#include <print.hpp>

#include <fstream>

int main(int argc, char** argv)
{
  std::ofstream file("log.txt");
  print(std::string("hello"), file);
}
EOF
```
```
edit README.md
```

Результат: Создание структуры проекта. Файл README.md отредактирован

### 7. Фиксация и отправка структуры проекта

```
git status
git add .
git commit -m "added sources"
git push origin main
```

Результат: Отправка второго коммита содержащий три новые директории
