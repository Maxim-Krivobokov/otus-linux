## Homework 9. Docker

Задача: Создать кастомный образ nginx на базе alpine linux. После запуска nginx должен отображать измененную страницу. 

* Образ создается командой docker build из dockerfile

```
docker build -t max89k/ainx:1.0
```

* в dockerfile за основу берется свежайший образ alpine (3.11). С помощью apk устанавливается nginx

* т.к в настройках по умолчанию nginx выводит только страницу 404, загрузим другой файл с настройкми, директивой COPY 

* в ней определена директория /var/lib/nginx/html/ , куда переместим кастомный html файл.
```
#copying config file to avoid 404 page
COPY ./default.conf /etc/nginx/conf.d/

# copy custom index.html
COPY ./index.html /var/lib/nginx/html/
```
* осталось прописать открытие порта 80, 443 (на всякий случай) и запустить nginx 

```
EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
```

Образ выгружен для общего доступа

````
docker push max89k/ainx1.2

# теперь доступен для скачивания 
docker pull max89k/ainx1.2
````
* Запуск контейнера (detached режим, проброшен порт 8081 на 80 в контейнер), проверка через curl или зайти в браузер (лучше не надо):
````
docker run -d -p 8081:80 max89k/ainx1.2 

curl 127.0.0.1:8081
````

### в чем отличие контейнера от образа

* image - своеобразный шаблон, из которого может быть создано множество контейнеров. Состоит из "замороженных" слоев, доутспных только для чтения.

* container - уникальный объект, созданный из образа. К нижним слоям образа добавляются один или несколько своих, с возможностью записи. 

Различия: 
- у контейнера есть состояния - 
- остановленный контейнер отличается от образа тем, что он сохраняет все изменения в параметрах настройки (напр, IP адрес),
  файлы в файловой системе, метаданные - все "незамороженные" уровни. 
 

 ### можно ли собрать ядро в контейнере

 Контейнер не имеет "своего" ядра, он обращается к ядру хостовой машины. Теоретически, скопировать и скомпилировать исходники можно, но процесс довольно бесполезный. 
 Пользоваться этим ядром, как виртуальная машина в HW №1, контейнер не сможет.