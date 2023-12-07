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
![variables page](img/VirtualBox_cibox_03_12_2023_15_22_42.png "variables page")

Pipeline passed
![pipeline passed](img/VirtualBox_cibox_03_12_2023_15_29_27.png "pipeline passed")

Deploy to preprod. На скриншоте — уникальные $MyLogin и $MyPassword для preprod
![deploy to preprod](img/VirtualBox_cibox_03_12_2023_15_31_48.png "deploy to preprod")

Deploy to production. Для production — свои $MyLogin и $MyPassword
![deploy to production](img/VirtualBox_cibox_03_12_2023_15_39_31.png "deploy to production")

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
![deploy to production](img/VirtualBox_cibox_03_12_2023_15_44_57.png "deploy to production")

