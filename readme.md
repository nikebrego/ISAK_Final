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

```
[nikita@localhost project]$ sudo yum install epel-release
```
Также нам необходимо установить pip диспетчер пакетов Python, и файлы разработки Python, необходимые для сборки uWSGI, незабудем также установить Nginx
```
[nikita@localhost project]$ sudo yum install python-pip python-devel gcc nginx  
```
#### Шаг второй. Процесс создания виртуальной среды Python
Для изоляции нашего приложения нам необходимо настроить виртуальную среду.
Для этого Установим пакет virtualenv:
```
[nikita@localhost project]$ sudo pip install virtualenv  
```
Создаем виртуальную среду
```
[nikita@localhost project]$ virtualenv projectvenv  
```
Далее нам необходимо активировать ее
```
[nikita@localhost project]$ source projectvenv/bin/activate  
```
После активации среды, поле ввода терминала приобретет следующий вид
```
(projectvenv) [nikita@localhost project]$  
```
#### Шаг третий. Создание, настройка приложения FLASK
Установка uwsgi и flask:
```
(projectvenv) [nikita@localhost project]$ pip install uwsgi flask  
```
Процесс создания приложения Flask:
```
(projectvenv) [nkita@localhost project]$ vi ~/project/app.py  
```
* Код приложения представлн в данном репозитории
Сохраним и закроем файл нажав ESC, а затем нажав Ctrl+Z.
Начнем тестирование приложения. Для этого нам необходимо запустить его в фоновом режиме:
```
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
```
(projectvenv) [nikita@localhost project]$ fg python app.py  
```
#### Шаг четверты. Создание точки входа WSGI
Создаем файл wsgi.py:
```
(projectvenv) [nikita@localhost project]$ vi ~/project/wsgi.py  
```
В тело файла необходимо вписать:
```
from project import app  
if __name__ == "__main__":  
  app.run()  
```
#### Шаг пятый. Настройка и конфигурация uWSGI
Для начала протестируем что uWSGI может обслуживать наше приложение:
```
(projectvenv) [nikita@localhost project]$ uwsgi --socket 0.0.0.0:8000 --protocol=http -w wsgi &  
```
Необходимо убедиться, что по указанному ранее адресу, но с портом 8000 находится содержимое html страницы. Выведем 5 первых строк.
```
(projectvenv) [nikita@localhost project]$ curl -L http://10.0.15:8000 | head -n 5  
```
![image](https://user-images.githubusercontent.com/56980417/121802114-a949e700-cc43-11eb-9fb3-97aea61d9fd8.png)
После этого приостановим uwsgi:
```
(projectvenv) [nikita@localhost project]$ fg  uwsgi --socket 0.0.0.0:8000 --protocol=http -w wsgi
```
Завершается работа с виртуальной средой. Выйдем из нее командой deactivate:
```
(projectvenv) [nikita@localhost project]$ deactivate  
```
Необходимо создать файл конфигурации uWSGI:
```
[nikita@localhost project]$ vi ~/project/project.ini  
```
В тело файла напишем:
```
[uwsgi]  
module = wsgi  
master = true  
processes = 3  
socket = project.sock  
chmod-socket = 660  
vacuum = true   
```
где "module = wsgi" - исполняемый модуль, созданный ранее файл "wsgi.py";

"master = true" - означает что uWSGI будет запускаться в главном режиме;

"processes = 3" - будет иметь 3 рабочих процесса для обслуживания запросов;

"socket = project.sock" - сокет, который будет использовать uWSGI;

"chmod-socket = 660" - права на процесс uWSGI;

"vacuum = true" - сокет будет очищен по завершении работы процесса;
#### Шаг шестой. Создание файла модуля systemd
Создадим файл службы:
```
[nikita@localhost project]$ sudo vi /etc/systemd/system/project.service   
```
В тело файла напишем:
```
[Unit]  
Description=uWSGI for project  
After=network.target  
[Service]  
User=nikita  
Group=nginx  
WorkingDirectory=/home/nikita/project  
Environment="PATH=/home/nikita/project/projectvenv/bin"  
ExecStart=/home/nikita/project/projectvenv/bin/uwsgi --ini project.ini  
[Install]  
WantedBy=multi-user.target  
```
где "[Unit]", "[Service]", "[Install]" - заголовки разделов;

"Description" - описание службы;

"After" - цель, по достижении которой будет производиться запуск;

"User" - пользователь;

"Group" - группа;

"WorkingDirectory" - рабочая директория в которой хранятся исполняемые файлы;

"Environment" - директория виртуальной среды;

"ExecStart" - команда для запуска процесса;

"WantedBy" - когда запускаться службе.

Запустим созданную службу:
```
[nikita@localhost project]$ sudo systemctl start project  
[nikita@localhost project]$ sudo systemctl enable project  
```
#### Шаг седьмой. Настройка NGINX
Теперь необходимо настроить Nginx для передачи веб-запросов в сокет с использованием uWSGI протокола.
Откроем файл конфигурации Nginx:
```
[nikita@localhost project]$ sudo vi /etc/nginx/nginx.conf  
```
Необходимо найти блок server {} в теле http:
```
...  
  http {  
  ...  
    include /etc/nginx/conf.d/*.conf;  
    *
    server {  
        listen 80 default_server;  
        }  
  ...  
  }  
...  
```
Чуть выше создаем свой блок
```
server {  
  listen 80;  
  server_name 10.0.2.15;  
  
  location / {
    include uwsgi_params;
    uwsgi_pass unix:/home/nikita/project/project.sock;
}
```
Закроем и сохраним файл.
Добавим nginx пользователя в свою группу пользователей с помощью следующей команды:
```
[nikita@localhost project]$ sudo usermod -a -G nikita nginx  
```
Предоставим группе пользователей права на выполнение в домашнем каталоге:
```
[nikita@localhost project]$ chmod 710 /home/nikita
```
Запускаем Nginx:
```
[nikita@localhost project]$ sudo systemctl start nginx  
[nikita@localhost project]$ sudo systemctl enable nginx  
```
### Вывод
В ходе выполнения данной курсовой работы было создано приложение Flask в виртуальной средe, которое позволяет загружать файлы формата ".tif", а так же получать цветное изображение NDVI. 
Так же была создана и настроена точка входа WSGI, благодаря которой любой сервер приложений, поддерживающий WSGI, сможет взаимодействовать с приложением Flask. 
В итоге была создана служба systemd, необходимая для автоматического запуска uWSGI при загрузке системы. 
В заключении был настроен серверный блок Nginx.
