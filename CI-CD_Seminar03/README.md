# CI-CD_Seminar03

# CI/CD. Семинар 03. Continuous delivery и continuous deployment (непрерывная доставка и развертывание)

## Задача
1. Добавить 2 окружения "preprod" и "production"
2. Добавить отдельные deploy job для каждой среды
3. Добавить переменную "$MyLogin" внутри .gitlab-ci.yml, которая будет меняться в зависимости от среды.
4. Добавить переменную "$MyPassword" не используя .gitlab-ci.yml, которая так же будет меняться в зависимости от среды.
5. Добавить скрипт в .gitlab-ci.yml, который найдёт все запущенные pipeline по названии ветки(ref) и остановит их.



## Решение

### Пункты 1, 2, 3, 4

Добавляем окружение `preprod` и `production`. В них используем переменную `$MyLogin` (в настройках она будет иметь разные значения).
```yaml
deploy to preprod:
  stage: deploy
  variables:
    TARGET_ENV: preprod
    MyLogin: "Preprod ROMAN"
  script:
    - echo "Do your deploy here to ${TARGET_ENV}"
    - echo ${DB_SERVER}
    - echo "MyLogin"
    - echo $MyLogin
    - echo "MyPassword"
    - echo $MyPassword
  only:
    - main
  environment:
    name: preprod
```


```yaml
deploy to production:
  stage: deploy
  variables:
    TARGET_ENV: production
    MyLogin: "Production ROMAN"
  script:
    - echo "Do your deploy here to ${TARGET_ENV}"
    - echo ${DB_SERVER}
    - echo "MyLogin"
    - echo $MyLogin
    - echo "MyPassword"
    - echo $MyPassword
  only:
    - main
  environment:
    name: production
```

Настройки переменных. Здесь добавляем переменную `$MyPassword` через интерфейс. Она тоже будет различна для разных сред исполнения. Имеет смысл сделать ее Masked (но сейчас так делать не станем, чтобы увидеть корректный вывод переменных). Плюс здесь же добавляем `RUNNER_TOKEN` — он понадобиться для скрипта удаления из пункта 5.
![variables page](https://github.com/Ask1509/CI-CD/blob/f275ce51141a7f582fd51fdb0c9f9361465aef64/CI-CD_Seminar03/img/VirtualBox_cibox_42.png)

Pipeline passed
![pipeline passed](https://github.com/Ask1509/CI-CD/blob/9c00caa8636f18d8f9017153ca4f9bce7ea7a9b8/CI-CD_Seminar03/img/VirtualBox_cibox_27.png)

Deploy to preprod. На скриншоте — уникальные $MyLogin и $MyPassword для preprod
![deploy to preprod](https://github.com/Ask1509/CI-CD/blob/55dacb75b918de02a4d0da22c066d97488b29520/CI-CD_Seminar03/img/VirtualBox_cibox_48.png)

Deploy to production. Для production — свои $MyLogin и $MyPassword
![deploy to production](https://github.com/Ask1509/CI-CD/blob/01dc770f1dbdf47bffb7bd48e852f6e3e1f09103/CI-CD_Seminar03/img/VirtualBox_cibox_31.png)

### Пункт 5

```yaml
# ------- Cancel -------
cancel:
  stage: stop previous jobs
  image: everpeace/curl-jq
  script:
    - |
      if [ "$CI_COMMIT_REF_NAME" == "main" ]
        then
          (
            echo "Cancel old pipelines from the same branch except last"
            OLD_PIPELINES=$( curl -s -H "PRIVATE-TOKEN: $RUNNER_TOKEN" "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/pipelines?ref=${CI_COMMIT_REF_NAME}&status=running" \
                  | jq '.[] | .id' | tail -n +2 )
                  for pipeline in ${OLD_PIPELINES}; \
                      do echo "Killing ${pipeline}" && \
                        curl -s --request POST -H "PRIVATE-TOKEN: ${RUNNER_TOKEN}" "https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/pipelines/${pipeline}/cancel"; done
          ) || echo "Canceling old pipelines (${OLD_PIPELINES}) failed"
      fi
```


Тут надо разбираться. Pipeline passed, но строки с 34 — вызывают сомнения.
![deploy to production](https://github.com/Ask1509/CI-CD/blob/a6110642d63349f59c19fca9dfec94a25fb9494b/CI-CD_Seminar03/img/VirtualBox_cibox_57.png)

