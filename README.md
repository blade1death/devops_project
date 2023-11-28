

# Проект CI/CD системы курса "DevOps практики и инструменты"
## Описание моего проекта пока рбезрезультатно
Для создания CI/CD системы было выдано простое микросервисное приложение:  
 - https://github.com/express42/search_engine_crawler
 - https://github.com/express42/search_engine_ui  

На основании данного приложения был выстроен процесс развертывания инфраструктуры, процесс CI, процесс CD.
## Иструменты и технологии
В проекте в той или иной мере используются:
- GCP, в частности GKE как платформа
- Terraform для развертывания кластера Kubernetes
- Kubernetes для создания условий функционирования микросервисов
- Helm для деплоя некоторых инфраструктурных приложений
- Docker-compose для тестирования работы приложения в локальном окружении
- Docker
- Travis-CI + некоторая обертка bash как основная CI/CD система
- EFK-stack (Elasticsearch, fluentd, Kibana) для создания системы логирования.
- Prometheus + Grafana для сбора и визуализации метрик
- MongoDB как БД для хранения данных приложения search_engine
- RabbitMQ как менеджер очередей для приложения search_engine
## Схема работы проекта
После развертывания инфраструктуры в GKE вы можете приступить к работе (доработке) с приложеним. По умолчанию, при создании Kubernetes-кластера создается namespace prod, в котором находится основная ветка приложения (master), при создании любых других веток в репозитории git и отправке изменений в git будет создаваться отдельное окружение (namespace) с названием ветки git. После слияния изменений с веткой master все изменения приложений переносятся в окружение prod. Все dev-окружения необходимо удалять **вручную**.
## Требования к запуску проекта
- Аккаунт в GCP
- Google Cloud SDK
- kubectl
- terraform
- docker (опционально, аккаунт на docker hub)
- (опционально) Установленный и настроенный docker-compose
## Как запустить проект
### Создание инфраструктуры
Запустить файл cluster.sh с параметром create или recreate:
```
bash cluster.sh create
```
Если запустить скрипт cluster.sh без параметров, будет показана справка.
**ВАЖНО:** 
 - перед запуском скрипта убедитесь, что у вас заданы переменные **SLACKAPIURL** и **SLACKCHANNEL**.
**SLACKAPIURL** - переменная с URL для API slack  
**SLACKCHANNEL** - название канала в slack  
Эти переменные нужны для алертинга от alertmanager. Если вы не хотите использовать алертинг, закомментируйте строки в файле cluster.sh, либо задайте переменным любое мусорное значение
 - перед запуском скрипта убедитесь, что вы находитесь в ветке **master**
Результаты работы скрипта cluster.sh:  
- Создание кластера kubernetes из 2х нод
- Удаление (**полностью!**) ~/.kube/config
- Создание нового ~/.kube/config с контекстом нового кластера
- Создание неймспейсов **prod**, **logging**, **monitoring**
- Деплой в неймспейс prod приложения вместе с MongoDB и RabbitMQ
- Установка и настройка helm
- Деплой Grafana и Prometheus в неймспейс monitoring. Оба приложения уже настроены в связке, есть нужные дашборды
- Деплой EFK-stack в неймспейс logging
- Вывод всех актуальных данных кластера: IP для доступа к web-интерфейсу приложения, готовые команды для port-forward приложений на локальный компьютер
### Работа с приложением
Доступ к приложению можно получить по указанному в предыдущем пункте IP.
Если у вас есть необходимость изменить/доработать приложение, просто создайте новую ветку (в имени ветки из спецсимволов допустимо использовать только "-"). Внесите изменения, запушьте новую ветку в Git-репозиторий.  
Исходные коды приложения находятся в:
- docker/search_engine_crawler
- docker/search_engine_ui  

Travis-ci проведет необходимые тесты приложения, если все в порядке, создаст новый namespace (именно с именованием namespace в kubernetes связано ограничение используемых символов в названиях веток), в который задеплоит новую версию приложения. Также, Travis-CI выведет IP, по которому можно посмотреть приложение. В проитивном случае, можете посмотреть IP-адрес в разделе Services в GKE.
После слияния feature-ветки в master вам нужно будет самостоятельно удалить feature-namespace в kubernetes.
Иными словами, при пуше изменений в ветку master меняется приложение в namespace **prod**.  
При пуше изменений в любую другую ветку создается/меняется приложение в namespace с названием этой ветки.
### Локальная работа с приложением
Для теста приложения на локальном компьютере используйте **docker_compose/docker-compose.yml**.
