# Краткий справочник

Источник: [WSargent's Docker Cheat Sheet](https://github.com/wsargent/docker-cheat-sheet)

## Образы

**[docker images](https://docs.docker.com/engine/reference/commandline/images)** показывает все образы.

**[docker build](https://docs.docker.com/engine/reference/commandline/build)** создаёт образ из файла `Dockerfile`.

**[docker commit](https://docs.docker.com/engine/reference/commandline/commit)** создает образ из контейнера, останавливает его, если он запущен.

**[docker rmi](https://docs.docker.com/engine/reference/commandline/rmi)** удаляет образ.

**[docker history](https://docs.docker.com/engine/reference/commandline/history)** показывает историю образа.

**[docker tag](https://docs.docker.com/engine/reference/commandline/tag)** создает тег для образа (локальному или в реестре).

## Жизненный цикл контейнеров

**[docker create](https://docs.docker.com/engine/reference/commandline/create)** создает контейнер, но не запускает его.

**[docker run](https://docs.docker.com/engine/reference/commandline/run)** создает и запускает контейнер за одну операцию.

**[docker rm](https://docs.docker.com/engine/reference/commandline/rm)** удаляет контейнер.

## Запуск и остановка контейнеров

**[docker start](https://docs.docker.com/engine/reference/commandline/start)** запускает контейнер, чтобы он работал.

**[docker stop](https://docs.docker.com/engine/reference/commandline/stop)** останавливает запущенный контейнер.

**[docker restart](https://docs.docker.com/engine/reference/commandline/restart)** останавливает и запускает контейнер.

**[docker pause](https://docs.docker.com/engine/reference/commandline/pause/)** приостанавливает работу контейнера, «замораживая» его на месте.

**[docker unpause](https://docs.docker.com/engine/reference/commandline/unpause/)** возобновляет работу запущенного контейнера.

**[docker exec](https://docs.docker.com/engine/reference/commandline/exec)** выполняет команду в контейнере.

**[docker attach](https://docs.docker.com/engine/reference/commandline/attach)** подключается к запущенному контейнеру.

## Информация о контейнере

**[docker ps](https://docs.docker.com/engine/reference/commandline/ps)** показывает запущенные контейнеры.

**[docker logs](https://docs.docker.com/engine/reference/commandline/logs)** получает логи из контейнера.

**[docker inspect](https://docs.docker.com/engine/reference/commandline/inspect)** показывает всю информацию о контейнере.

**[docker port](https://docs.docker.com/engine/reference/commandline/port)** показывает открытый порт контейнера.

**[docker stats](https://docs.docker.com/engine/reference/commandline/stats)** показывает статистику использования ресурсов контейнеров.

**[docker diff](https://docs.docker.com/engine/reference/commandline/diff)** показывает измененные файлы в файловой системе контейнера.
