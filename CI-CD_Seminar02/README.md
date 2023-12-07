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
![docker container test](img/VirtualBox_cibox_30_11_2023_11_42_19.png "docker container test")

Просматриваем список созданных контейнеров (ишь, расплодились!).
![list of containers](img/VirtualBox_cibox_30_11_2023_11_44_11.png "list of containers")

Pipeline passed
![pipeline passed](img/VirtualBox_cibox_30_11_2023_11_44_42.png "pipeline passed")

## Замечание
При создании образа (раздел `docker build:`) — появлялась проблема нехватки прав.

`Got permission denied while trying to connect to the Docker daemon socket at… `

Исправляется добавлением gitlab-runner в группу docker.

```
usermod -aG docker gitlab-runner
```