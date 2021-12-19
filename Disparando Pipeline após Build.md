
#### Etapa para disparar outra pipeline quando o build for concluido
 
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

#### Nessa etapa da pipeline é executado um scripts Python que recebe os argumentos:

- *TOKEN CODEFRESH*
- *NOME DO PROJETO*
- *BRANCH*
- *NOME DO AMBIENTE*
  **Ex:** 
    - *dev, hml ou prod





#### O seguinte script é executado :

```python
headers = {'Authorization': sys.argv[1]}

pipelineName =  sys.argv[2] + "%2F" +  sys.argv[2] + "-" +  sys.argv[4]

data = {
  "branch": sys.argv[3],
  "serviceName": pipelineName,
  "type": "build",
  "variables": {
  },
  "options": {
    "noCache": "false",
    "noCfCache": "false",
    "resetVolume": "false",
    "enableNotifications": "true"
  },
  "isYamlService": "true",
  "trigger": ""
}

print(pipelineName)

pipeline = requests.get("https://g.codefresh.io/api/pipelines/" + pipelineName, headers=headers)

if pipeline.status_code == 200:
  pipeline_json = pipeline.json()

  print(pipeline_json["spec"]["triggers"][0]["id"])

  data["trigger"] = pipeline_json["spec"]["triggers"][0]["id"]

  response_run_pipeline = requests.post("https://g.codefresh.io/api/pipelines/run/" + pipelineName, json=data, headers=headers)

  print(response_run_pipeline.status_code)
```

**O script dispara uma chamada via api para o codefresh que aciona outra pipeline**

**Obs**:para que o script funcione corretamente é necessario 
seguir determinado padrao no nome da PASTA CONTENDO AS PIPELINES, NOME DO PROJETO e NOME DAS PIPELINES



**Com esses padroes atendidos:**

É concatenado NOME DO PROJETO +"%2F"+ NOME DO PROJETO + "-" + NOME DO AMBIENTE
e definido o caminho da pipeline a ser disparada.

O processo para disparar a pipeline é feito via API do Codefresh, com isso os itens concatenados sao usados como parte do endpoint e tambem no corpo da requisição
  Ex:
  pipelineName = NOME DO PROJETO +"%2F"+ NOME DO PROJETO + "-" + NOME DO AMBIENTE
  https://g.codefresh.io/api/pipelines/" + pipelineName






