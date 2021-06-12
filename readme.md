# ISAK Курсовая работа
## Быков Никита М3О-312Б-18.
### Постановка задачи
1. Скачать и установить веб-сервер.
2. Настроить его на работу с localhost.
3. Реализовать форму с загрузкой файла.
4. Захостить python приложение из предыдущего семестра, при загрузке снимка рисовать в веб карту NDVI.
### Ход выполнения работы:
#### Шаг первый. Установка компонентов
Устанавливаем репозиторий EPEL, который содержит дополнительные пакеты.

```sh
[nikita@localhost project]$ sudo yum install epel-release
```
Также нам необходимо установить pip диспетчер пакетов Python, и файлы разработки Python, необходимые для сборки uWSGI, незабудем также установить Nginx
```sh
[nikita@localhost project]$ sudo yum install python-pip python-devel gcc nginx  
```
#### Шаг второй. Процесс создания виртуальной среды Python
Для изоляции нашего приложения нам необходимо настроить виртуальную среду.
Для этого Установим пакет virtualenv:
```sh
[nikita@localhost project]$ sudo pip install virtualenv  
```
Создаем виртуальную среду
```sh
[nikita@localhost project]$ virtualenv projectvenv  
```
Далее нам необходимо активировать ее
```sh
[nikita@localhost project]$ source projectvenv/bin/activate  
```
После активации среды, поле ввода терминала приобретет следующий вид
```sh
(projectvenv) [nikita@localhost project]$  
```
#### Шаг третий. Создание, настройка приложения FLASK

