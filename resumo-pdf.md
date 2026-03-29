UFO Tracker - Relatorio do Projeto


Missao 1 - Banco de Dados PostgreSQL

Foi criado o namespace "ufology" no Kubernetes para isolar os recursos da operacao. Dentro dele, um Deployment com 1 replica foi configurado usando a imagem customizada leogloriainfnet/ufodb:latest, com as variaveis POSTGRES_USER, POSTGRES_PASSWORD e POSTGRES_DB definidas conforme solicitado. Um Service do tipo ClusterIP expoe o banco na porta 5432 internamente no cluster.

Comando para aplicar:
kubectl apply -f k8s/ufology.yaml


Missao 2 - Redis (Cache)

Um Deployment do Redis foi criado no mesmo namespace ufology, com 1 replica usando a imagem redis:7-alpine. Um Service ClusterIP na porta 6379 permite que outros pods acessem o cache. Labels foram usadas para conectar o Service aos Pods corretos.


Missao 3 - Dockerizacao da Aplicacao

O projeto Java (Spring Boot 3.5 + Java 21) foi clonado do repositorio do professor. Um Dockerfile multi-stage foi criado:
- Stage 1 (build): usa eclipse-temurin:21-jdk-alpine, copia o codigo e roda mvnw package
- Stage 2 (runtime): usa eclipse-temurin:21-jre-alpine, copia o JAR e expoe a porta 8080

Comandos utilizados:
docker build -t evertonrocha2/ufo-tracker:latest .
docker push evertonrocha2/ufo-tracker:latest

Link da imagem: https://hub.docker.com/r/evertonrocha2/ufo-tracker


Missao 4 - Deploy da Aplicacao no Cluster

Foram criados:
- ConfigMap "app-config" com DB_NAME=ufology
- Secret "db-secret" com DB_PASSWORD=devops2025!
- Deployment "ufo-tracker" com 2 replicas, consumindo o ConfigMap e Secret via env, conectando ao postgres e redis pelos Services internos
- Service ClusterIP na porta 8080 para a aplicacao

O application.yaml do Spring Boot usa as variaveis POSTGRES_USERNAME e POSTGRES_PASSWORD para conexao, e o Redis aponta para o host "redis" na porta 6379.


Parte 2 - Workflows Basicos

hello.yml: roda em qualquer push, executa echo "Hello CI/CD"
tests.yml: roda em pull_request, faz checkout e executa echo "Rodando testes"
gradle-ci.yml: roda em push para main, configura JDK 21, roda testes, faz build com Maven e usa upload-artifact para armazenar o JAR gerado


Parte 3 - Runners, Variaveis e Seguranca

env-demo.yml: define DEPLOY_ENV=staging no nivel do workflow e exibe no log
secret-demo.yml: usa secrets.API_KEY e exibe "API_KEY configurado" sem expor o valor
run-monitor.yml: demonstra variaveis em 3 niveis (workflow, job e step), usa vars do repositorio e mostra uso do GITHUB_TOKEN
deploy.yml: pipeline com 3 jobs (test > validate > deploy), valida variaveis obrigatorias antes de prosseguir, exige aprovacao manual via environment "production", usa PROD_DOMAIN como variavel do ambiente protegido
debug-monitor.yml: habilita ACTIONS_STEP_DEBUG, usa ::warning:: e ::error:: para mensagens customizadas, gera job summary com tabela de resultados, inclui diagnostico automatizado de falhas com sugestoes de correcao


Diferenca entre Runners hospedados e auto-hospedados

Runners hospedados pelo GitHub sao maquinas virtuais gerenciadas pelo proprio GitHub. Vantagens: nao precisam de manutencao, ambiente limpo a cada execucao, varios sistemas operacionais disponiveis. Desvantagens: tempo de execucao limitado a 6 horas, hardware padrao sem customizacao, sem acesso a redes privadas.

Runners auto-hospedados (self-hosted) sao maquinas proprias configuradas pelo usuario. Vantagens: hardware customizado, acesso a recursos internos e redes privadas, sem limite de tempo do GitHub, ferramentas pre-instaladas. Desvantagens: manutencao e seguranca por conta do usuario, custo de infraestrutura, risco de ambiente contaminado entre execucoes se nao houver limpeza adequada.


Configuracoes necessarias no GitHub

Antes de rodar os workflows, configure no repositorio:
1. Settings > Secrets and variables > Actions > New repository secret:
   - API_KEY: qualquer valor (ex: minha-chave-secreta)
   - DB_PASSWORD: devops2025!
2. Settings > Secrets and variables > Actions > Variables:
   - DEPLOY_ENV: staging
   - PROD_DOMAIN: ufo-tracker.example.com
3. Settings > Environments > New environment:
   - Nome: production
   - Marcar "Required reviewers" e adicionar seu usuario
