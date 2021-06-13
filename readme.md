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
Установка uwsgi и flask:
```sh
(projectvenv) [nikita@localhost project]$ pip install uwsgi flask  
```
Процесс создания приложения Flask:
```sh
(projectvenv) [nkita@localhost project]$ vi ~/project/app.py  
```
* Код приложения представлн в данном репозитории
Сохраним и закроем файл нажав ESC, а затем нажав Ctrl+Z.
Начнем тестирование приложения. Для этого нам необходимо запустить его в фоновом режиме:
```sh
(projectvenv) [nikita@localhost project]$ python ~/project/app.py &   
```
> *Serving Flask app 'app' (lazy loading)
> 
> *Environment: prodction
> 
> *Debug mode: on
> 
> *Running on all addresses.
> 
> *Running on http://10.0.15:5000 (Press CTRL+C) to quit

Далее останавливаем Flask приложение с помощью fg:
```sh
(projectvenv) [nikita@localhost project]$ fg python app.py  
```
#### Шаг четверты. Создание точки входа WSGI
Создаем файл wsgi.py:
```sh
(projectvenv) [nikita@localhost project]$ vi ~/project/wsgi.py  
```
В тело файла необходимо вписать:
```sh
from project import app  
if __name__ == "__main__":  
  app.run()  
```
#### Шаг пятый. Настройка и конфигурация uWSGI
