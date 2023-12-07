# CI/CD. Семинар 04. Troubleshooting (диагностика и решение проблем в CI/CD)

## Задача
Сделать локальный шаблон CI и отдельный репозиторий с шаблонами, подключить их к своему основному репозиторию через include




## Решение
Создан локальный файл `local-smoke-tests.gitlab-ci.yml`

```yaml
smoke-test-job:
  script: echo "SMOKE"
```

Создан основной файл `.gitlab-ci.yml`

```yaml
include:
  - local: local-smoke-tests.gitlab-ci.yml
#  - remote: https://github.com/DeleteDone/CI-CD/blob/main/CI-CD_Seminar04/remote_included-file.yml
```

Файл с аналогичным содержанием, что и у Максима, есть в этом репозитории: `remote_included-file.yml`



Repository Seminar04
![repository](https://github.com/DeleteDone/CI-CD/blob/cc385952cf4bed5396ed123a4e301f0f7aac2271/CI-CD_Seminar04/img/VirtualBox_cibox_47.png)

Remote included file job
![remote included file job](https://github.com/DeleteDone/CI-CD/blob/d1a9141ed78762ec6f63a29483ef7f9e43e1552a/CI-CD_Seminar04/img/VirtualBox_cibox_01.png)

Pipeline passed
![pipeline passed](https://github.com/DeleteDone/CI-CD/blob/49c518623b7b86f7c06772eb395a5bce88beae55/CI-CD_Seminar04/img/VirtualBox_cibox_34.png)

