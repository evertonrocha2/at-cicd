![CI Status](https://img.shields.io/github/actions/workflow/status/evertonrocha2/at-cicd/gradle-ci.yml?branch=main&label=CI)

# UFO Tracker

Sistema de rastreamento de avistamentos UFO da divisao Ufology Investigation Unit.

## O papel do Git na entrega continua

O Git e a base de qualquer pipeline de CI/CD. Ele permite que cada alteracao no codigo seja rastreada, versionada e auditada. Quando um desenvolvedor faz um push, o GitHub Actions detecta a mudanca e dispara os workflows automaticamente, como build, testes e deploy. Sem controle de versao, nao existe integracao continua, porque nao ha como saber o que mudou, quando mudou ou quem mudou.

No ciclo DevOps, o Git conecta o trabalho do desenvolvedor com a automacao de infraestrutura. O codigo commitado vira o gatilho para todo o pipeline: testes automaticos verificam se nada quebrou, o artefato e gerado e o deploy pode ser feito de forma automatica ou com aprovacao manual. O historico de commits serve como documentacao viva do projeto.

## Branches e tags no contexto de CI/CD

Branches permitem que funcionalidades sejam desenvolvidas de forma isolada sem afetar a branch principal. No CI/CD, isso e importante porque cada branch pode ter seu proprio pipeline de validacao. A branch main representa o codigo estavel e pronto para producao, enquanto branches como ci/setup ou feature/xyz sao usadas para desenvolvimento.

Tags marcam versoes especificas do codigo (ex: v1.0.0). No contexto de entrega continua, tags sao usadas para disparar releases e deploys de versoes especificas. Elas criam pontos fixos no historico que podem ser referenciados para rollback ou auditoria. Uma tag garante que voce sempre pode voltar a uma versao exata do sistema.

## Stack

- Java 21 + Spring Boot 3.5
- PostgreSQL (banco de dados)
- Redis (cache)
- Maven (build)
- Docker + Kubernetes (infraestrutura)
- GitHub Actions (CI/CD)

## Como rodar

```bash
# build da imagem
docker build -t evertonrocha2/ufo-tracker:latest .

# aplicar manifestos k8s
kubectl apply -f k8s/ufology.yaml
```

## Workflows

| Workflow | Trigger | Descricao |
|----------|---------|-----------|
| hello.yml | push | Exibe "Hello CI/CD" |
| tests.yml | pull_request | Roda testes |
| gradle-ci.yml | push main | Build Maven + upload artifact |
| env-demo.yml | push | Mostra variavel de ambiente |
| secret-demo.yml | push | Verifica secret API_KEY |
| run-monitor.yml | push | Demonstra variaveis em niveis diferentes |
| deploy.yml | push main | Pipeline com aprovacao manual |
| debug-monitor.yml | push | Debug, warnings, errors e job summary |
