# Лабораторная работа №4

## Обязательное задание

### Введение

В данной лабораторной работе в качестве CI/CD инструмента используется GitHub Actions, для которого были написаны 2 `.yml` файла `CI/CD with Best Practices` и `CI/CD with Bad Practices`.

### Bad Practices

**1. Затрагивает все ветки и все директории для пушей**

CI/CD пайплайн настроен, чтобы взаимодействовать с проектом находящим в lab-4. Однако без спецификации ветки и директории workflow будет запускаться при взаимодействии с любой веткой и директорией в репозитории `itmo-clouds-labs`.

```
on:
    push:
        branches:
            - '*'
```

**2. Не указана версия Python**

Из-за этого поведение кода может отличаться, а также многие библиотеки не смогут подгрузиться, т.к. требуют актуальную версию Python.

**3. Неявное указание каталога тестов**

Мы хотим, чтобы использовалась конкретная директория tests, в которой находятся тесты, однако данный степ будет искать тесты по всей директории с кодом, а не в конкретной указанной разработчиком.

```
- name: Run all tests
    run: |
        cd labs/lab-4
        python -m unittest discover
```

**4. Присутствует степ для деплоя без проверок**

Необходимо добавлять шаги для проверки. Например, использовать `if: success()` для проверки статуса тестов. \\
(В данном случае в качестве деплоя стоит заглушка в виде вывода `Deploying to production...`, поэтому ничего плохого не произойдёт, если данная строка напечатается даже при сценарии, когда тесты не выполняются. В реальном проекте это может быть фатальной ошибкой)

```
- name: Build and deploy to production
    run: |
        echo "Deploying to production..."
```

**5. Неэффективные уведомления**

Все шаги должны зависеть от успешного завершения сборки, чтобы избежать ненужных уведомлений. Однако в данном случае job с уведомлением запустится в любом случае (в не зависимости от результата выполнения сборки).

```
notify:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Notify failure
            run: |
                if [ "${{ job.status }}" == "failure" ]; then 
                echo "Build failed! Notifying team..." 
                fi
```

### Исправление Bad Practices

**1. Выбраны рабочие директории и ветки**

Пайплайн затрагивает только директории labs/lab-4 и .github/workflows, а также только ветку lab-4.

```
on:
  push:
    branches:
      - lab-4
    paths:
      - 'labs/lab-4/**'
      - '.github/workflows/good-lab-4-ci-cd.yml'
  pull_request:
    branches:
        - lab-4
    paths:
        - 'labs/lab-4/**'
        - '.github/workflows/good-lab-4-ci-cd.yml'
```

**2. Указана версия Python**

Чтобы зафиксировать поведение кода была выбрана версия Python 3.10.

```
- name: Set up Python
    uses: actions/setup-python@v2
    with:
        python-version: '3.10'
```

**3. Явное указание директории с тестами**

Теперь при запуске workflow запускаются только тесты из директории tests во избежание поиска тестов в других файлах и директориях.

```
- name: Run tests
    run: |
        cd labs/lab-4
        python -m unittest discover -s tests -p "*.py"
```

**4. Добавлена проверка выполнения тестов перед деплоем**

Перед деплоем требуется как минимум выполнение всех указанных тестов, поэтому перед этим шагом необходимо устанавливать проверку на выполнение тестов.

```
- name: Build and deploy to production
    if: success()
    run: echo "Deploying to production..."
```

**5. Эффективные уведомления**

Теперь уведомления будут запускаться только при условии, что во время выполнения workflow не возникло ошибок

```
notify:
    runs-on: ubuntu-latest
    needs: build
    if: failure()
    steps:
      - name: Notify failure
        run: |
          echo "Build failed! Notifying team..."
```