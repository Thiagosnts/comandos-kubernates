![PipeLine-build](resources/pipeline-build.png)
```yaml
version: '1.0'
```
#### Listando todas as etapas
```yaml
stages:
  - Clean
  - Checkout
  - QA
  - BuildImage
  - Deploy
  - Notification
```
```yaml
steps:
```
#### Atribuindo imagem e limpando referências usadas anteriormente 

```yaml
  clean:
    fail_fast: false
    stage: Clean
    image: alpine:latest
    commands:
      - rm -rf ${{app_name}}
      - rm -rf scripts
      - rm -rf sonar_app_config
```

#### Obtendo a aplicação do repositório
```yaml
  checkout_project:
    fail_fast: false
    stage: Checkout
    image: alpine/git:1.0.7
    commands:
      - git clone https://oauth2:${{gitlab_codefresh_token}}@${{repo_app}} ${{app_name}} --branch ${{CF_BRANCH}}
    when:
      steps:
        - name: clean
          on:
            - success
```

#### Entrando na pasta do projeto e fazendo substuição de um arquivo de configuração caso seja para ambiente de develop.
**Obs:** *Etapa de uso especifico da Aplicação*
```yaml
  checkout_project_develop:
    fail_fast: false
    stage: Checkout
    image: alpine:latest    
    commands:
      - cd "${{app_name}}"/Api/Take.Api.Ipiranga
      - rm appsettings.json
      - mv appsettings.Development.json appsettings.json  
    when:
      condition:
          all:
            isBranchDevelop: "'${{CF_BRANCH}}' == 'develop' && steps.checkout_project.result == 'success'"
```
#### Obtendo [scripts](https://gitlab.ipirangacloud.com/devops/scripts/scripts-codefresh) no repositório para uso de funções durante as etapas da pipeline, as funcoes contidas nesse repositórios são:
- *Notification*
- *Disparo de outra pipeline*
- *Health check do serviço*
- *Sonar*

```yaml
  checkout_scripts:
    fail_fast: false
    stage: Checkout
    image: alpine/git:1.0.7
    commands:
      - git clone https://oauth2:${{gitlab_codefresh_token}}@${{repo_scripts_pipeline}} scripts
    when:
      steps:
        - name: checkout_project
          on:
            - success

```
#### Obtendo Arquivo no repositório com propiedades para o uso do sonar
```yaml
  checkout_sonar_app_config:
    fail_fast: false
    stage: Checkout
    image: alpine/git:1.0.7
    commands:
      - git clone https://oauth2:${{gitlab_codefresh_token}}@${{repo_sonar_app_config}} sonar_app_config
    when:
      steps:
        - name: checkout_scripts
          on:
            - success
```
#### Copiando o arquivo com as propiedades do sonar para pasta do projeto atual e substuindo nesse arquivo os dados para o contexto necessário.
```yaml
  prepare_sonar:
    fail_fast: false
    stage: QA
    image: alpine:latest
    commands:
      - cp sonar_app_config/sonar-project.properties ${{app_name}}/
      - cd ${{app_name}}
      - sed -i -e 's/${sonar_host}/${{sonar_host}}/g' sonar-project.properties
      - sed -i -e 's/${sonar_token}/${{sonar_token}}/g' sonar-project.properties
      - sed -i -e 's/${app_name}/${{app_name}}/g' sonar-project.properties
    when:
      steps:
        - name: checkout_sonar_app_config
          on:
            - success
```            
#### Atribuindo imagem com dependencia necessária para o projeto atual
```yaml
  run_unit_tests:
    fail_fast: false
    stage: QA
    image: mcr.microsoft.com/dotnet/core/sdk:3.1
    commands:   
      - cd ${{app_name}}/Api
      - dotnet test
    when:
      condition:
        any:
          isToExecuteUnitTests: "'${{IS_EXECUTE_UNIT_TESTS}}' == 'true'" 
```   
#### Atribuindo imagem com sonar e atribuindo pasta do projeto
```yaml
  run_sonar:
    fail_fast: false
    stage: QA
    image: newtmitch/sonar-scanner:4.0.0
    volumes:
      - ./${{app_name}}:/srv
    commands:
      - sonar-scanner -Dsonar.projectBaseDir=/srv
    when:
      steps:
        - name: run_unit_tests
          on:
            - success
            - skipped
```
#### Executando scripts carregado nos passos anterioes e testando o sonar

```yaml
  check_sonar_quality_gate:
    fail_fast: false
    stage: QA
    image: python:3.9.0a3-alpine3.10
    commands:
      - pip3 install requests
      - python3 scripts/check-sonar-quality-gate.py "${{app_name}}" "${{sonar_token}}"
    when:
      steps:
        - name: run_sonar
          on:
            - success
```            
#### Criando imagem com Dockfile
```yaml
  build_image:
    fail_fast: false
    stage: BuildImage
    type: build
    working_directory: ./${{app_name}}/Api
    dockerfile: Dockerfile
    image_name: ${{app_name}}
    tag: ${{CF_SHORT_REVISION}}
    registry: ${{registry_name}}
    when:
      steps:
        - name: check_sonar_quality_gate
          on:
            - success
```

```yaml
  deploy_dev:
    fail_fast: false
    stage: Deploy
    image: python:3.9.0a3-alpine3.10
    commands:
      - pip3 install requests
      - python3 scripts/run_pipeline.py "${{codefresh_api_token}}" "${{app_name}}" "${{CF_BRANCH}}" "dev"
    when:
      condition:
        all:
          isToExecuteDeployDev: "'${{CF_BRANCH}}' == 'develop' && steps.build_image.result == 'success'"
```
#### Executando scripts carregado nos passos anterioes e disparando outra pipeline de QA
```yaml
  deploy_qa:
    fail_fast: false
    stage: Deploy
    image: python:3.9.0a3-alpine3.10
    commands:
      - pip3 install requests
      - python3 scripts/run_pipeline.py "${{codefresh_api_token}}" "${{app_name}}" "${{CF_BRANCH}}" "qa"
    when:
      condition:
        all:
          isToExecuteDeployDev: "'${{CF_BRANCH}}' == 'qa' && steps.build_image.result == 'success'"
```
#### Executando scripts carregado nos passos anterioes e disparando outra pipeline de HML
```yaml
  deploy_hml:
    fail_fast: false
    stage: Deploy
    image: python:3.9.0a3-alpine3.10
    commands:
      - pip3 install requests
      - python3 scripts/run_pipeline.py "${{codefresh_api_token}}" "${{app_name}}" "${{CF_BRANCH}}" "hml"
    when:
      condition:
        all:
          isToExecuteDeployHml: "'${{CF_BRANCH}}' == 'release' && steps.build_image.result == 'success'"
```
#### Executando scripts carregado nos passos anterioes e disparando outra pipeline de prod
```yaml
  deploy_prd:
    fail_fast: false
    stage: Deploy
    image: python:3.9.0a3-alpine3.10
    commands:
      - pip3 install requests
      - python3 scripts/run_pipeline.py "${{codefresh_api_token}}" "${{app_name}}" "${{CF_BRANCH}}" "prd"
    when:
      condition:
        all:
          isToExecuteDeployPrd: "'${{CF_BRANCH}}' == 'master' && steps.build_image.result == 'success'"
```
#### Faz a checagem se todas as etapas tiveram sucesso, em caso positivo executa os scripts carregados nos passos anterioes e dispara uma notificação por email sobre o sucesso do processo
```yaml
  notification_success:
    stage: Notification
    image: python:3.9.0a3-alpine3.10
    commands:
      - pip3 install sendgrid
      - python3 scripts/notification/build/notification_success.py "${{app_name}}" "${{CF_REVISION}}" "${{CF_COMMIT_MESSAGE}}" "${{emails}}"
    when:
      steps:
        all:
          - name: clean
            on:
              - success
          - name: checkout_project
            on:
              - success
          - name: checkout_scripts
            on:
              - success
          - name: checkout_sonar_app_config
            on:
              - success
          - name: prepare_sonar
            on:
              - success
          - name: run_unit_tests
            on:
              - success
              - skipped
          - name: run_sonar
            on:
              - success
          - name: check_sonar_quality_gate
            on:
              - success
          - name: build_image_dev
            on:
              - success          
          - name: build_image
            on:
              - success
          - name: deploy_dev
            on:
              - success
              - skipped
          - name: deploy_qa
            on:
              - success
              - skipped
          - name: deploy_hml
            on:
              - success
              - skipped
          - name: deploy_prd
            on:
              - success
              - skipped
```
#### Faz a checagem se alguma  etapa falhou, em caso positivo executa os scripts carregados nos passos anteriores e dispara uma notificação por email sobre o falha no processo
```yaml
  notification_failure:
    stage: Notification
    image: python:3.9.0a3-alpine3.10
    commands:
      - pip3 install requests
      - pip3 install sendgrid
      - python3 scripts/notification/build/notification_failure.py "${{codefresh_api_token}}" "${{CF_REVISION}}" "${{app_name}}" "${{CF_COMMIT_MESSAGE}}" "${{emails}}"
      - exit 1
    when:
      steps:
        any:
          - name: clean
            on:
              - failure
          - name: checkout_project
            on:
              - failure
          - name: checkout_scripts
            on:
              - failure
          - name: checkout_sonar_app_config
            on:
              - failure
          - name: prepare_sonar
            on:
              - failure
          - name: build_unit_tests_image
            on:
              - failure
          - name: run_unit_tests
            on:
              - failure
          - name: run_sonar
            on:
              - failure
          - name: check_sonar_quality_gate
            on:
              - failure
          - name: build_image_dev
            on:
              - failure
          - name: build_image
            on:
              - failure
          - name: deploy_dev
            on:
              - failure
          - name: deploy_qa
            on:
              - failure
          - name: deploy_hml
            on:
              - failure
          - name: deploy_prd
            on:
              - failure
          - name: notification_success
            on:
              - failure
```