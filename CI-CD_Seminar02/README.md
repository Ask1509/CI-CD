# CI/CD. Семинар 02. Continuous integration (непрерывная интеграция)

## Задача
Переписать test stage для тестирования docker

## Решение

После создания контейнера docker
```yaml
docker build:
  image: docker:latest
  stage: build
  services:
    - docker:dind
  script:
    - docker login -u $GITLAB_CI_USER -p $GITLAB_CI_PASSWORD $CI_REGISTRY
    - echo $GITLAB_CI_USER $GITLAB_CI_PASSWORD $CI_REGISTRY $CI_REGISTRY_IMAGE:$IMAGE_TAG
    - docker build -t $CI_REGISTRY_IMAGE:$IMAGE_TAG .
    - docker push $CI_REGISTRY_IMAGE:$IMAGE_TAG
```

проверяем возможность его загрузки из нашего реестра:
```yaml
testdocker:
  stage: test
  script:
    - docker pull $CI_REGISTRY_IMAGE:$IMAGE_TAG
    - docker images 
    - echo "----------MY DOCKER IMAGE TEST----------"
```

## Скриншоты
Добавляем блок тестирования (тянем образ контейнера).
![docker container test](https://github.com/Ask1509/CI-CD/blob/0386069191e2f1aa02a542414f3b40bd61fc428c/CI-CD_Seminar02/img/VirtualBox_ciibox19.png)

Просматриваем список созданных контейнеров (ишь, расплодились!).
![list of containers](https://github.com/Ask1509/CI-CD/blob/0386069191e2f1aa02a542414f3b40bd61fc428c/CI-CD_Seminar02/img/VirtualBox_cibox_11.png)

Pipeline passed
![pipeline passed](https://github.com/Ask1509/CI-CD/blob/3999c2711dc084a29ca57f63faba6ac1c41090f8/CI-CD_Seminar02/img/VirtualBox_cibox42.png)

## Примечание
При создании образа (раздел `docker build:`) — появлялась проблема нехватки прав.

`Got permission denied while trying to connect to the Docker daemon socket at… `

Исправляется добавлением gitlab-runner в группу docker.

```
usermod -aG docker gitlab-runner
```
