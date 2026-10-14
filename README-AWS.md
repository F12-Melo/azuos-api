# Preparação para AWS CodeBuild + CodeDeploy (ECS/ECR)

Este repositório foi ajustado para ser usado como base de build de imagem Docker para deploy na AWS.

Arquivos incluídos:
- `Dockerfile`: gera a imagem da API Flask com Gunicorn.
- `.dockerignore`: reduz o contexto do build.
- `buildspec.yml`: instruções para CodeBuild, incluindo login no ECR, build da imagem, push e geração dos artefatos.
- `appspec.yaml`: esqueleto para deploy ECS via CodeDeploy.
- `taskdef.json`: template de task definition do ECS com placeholders para substituição dos valores reais da AWS.

Observações importantes:
1. O app usa `gunicorn app:app` e expõe a porta 5000.
2. Para o deploy funcionar no ECS, você precisa criar/ajustar:
   - repositório ECR (`azuos-api` por padrão)
   - cluster ECS
   - serviço ECS com task definition e Load Balancer
   - role `ecsTaskExecutionRole`
   - role `azuos-api-task-role` se a API acessar outros serviços da AWS
3. No CodeBuild, configure as variáveis de ambiente:
   - `AWS_ACCOUNT_ID`
   - `AWS_DEFAULT_REGION`
4. Os valores de `taskdef.json` e `appspec.yaml` devem ser ajustados para os nomes reais do seu cluster/serviço/container.
5. Caso a API use banco de dados, configure `DATABASE_URL` e demais variáveis de ambiente no ECS, por exemplo via Secrets Manager ou variáveis de ambiente do serviço.

Exemplo de criação do repositório ECR:

```bash
aws ecr create-repository --repository-name azuos-api --region us-east-1
```

Exemplo de configuração do CodeBuild no console AWS:
- Source: GitHub
- Buildspec file: `buildspec.yml`
- Environment: privileged mode habilitado para Docker
- Role IAM com permissões para ECR e CodeBuild

Após o build, ajuste os arquivos AWS para os nomes reais do seu ambiente e aplique o deploy no ECS.
