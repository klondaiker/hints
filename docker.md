# Docker
## Команды Docker
- docker build - Собрать образ Docker (docker build .)
- docker tag - Присвоить тег образу Docker (docker tag 66c76cea05bb todoapp)
- docker run - Запустить образ Docker в качестве контейнера (docker run -i -t -p 8000:8000 --name example1 todoapp)
- docker commit - Сохранить контейнер Docker в качестве образа
- docker images - Список всех образов
- docker ps - Список всех контейнеров (docker ps -a)
- docker start - перезапустить контейнер (docker start example1)
- docker rm - очищает контейнер (docker rm example1)
- docker logs - просмотр вывода контейнера (docker logs example1)
- docker diff - какие файлы были затронуты с момента создания экземпляра образа как контейнера (docker diff example1)
## Dockerfile
```Dockerfile
# Определяет базовый образ
FROM node
# Объявляет автора
LABEL maintainer klondaiker@bk.ru
# Клонирует код приложения
RUN git clone -q https://github.com/docker-in-practice/todo.git
# Перемещает в новый клонированный каталог
WORKDIR todo
# Запускает команду установки менеджера пакетов (npm)
RUN npm install > /dev/null
# Указывает, что контейнеры из собранного образа должны прослушивать этот порт
EXPOSE 8000
# Указывает, какая команда будет запущена при запуск
CMD ["npm", "start"]
```
