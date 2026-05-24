# Agente de Investimentos no n8n

<img width="946" height="293" alt="image" src="https://github.com/user-attachments/assets/07fd7953-c435-48b0-b0f6-54f61dde56e8" />


Projeto de automacao para monitorar uma carteira de ativos brasileiros e `Bitcoin`, gerar uma analise diaria com IA e enviar o relatorio por email em dias uteis.

Arquivos principais:

- [n8n-workflow-analise-ativos.json](/Users/jacquesjacob/Documents/agente-investimentos/n8n-workflow-analise-ativos.json)
- [docker-compose.yml](/Users/jacquesjacob/Documents/agente-investimentos/docker-compose.yml)
- [.env.example](/Users/jacquesjacob/Documents/agente-investimentos/.env.example)
- [README-docker-n8n.md](/Users/jacquesjacob/Documents/agente-investimentos/README-docker-n8n.md)

## Visao geral

Este workflow foi montado para:

- rodar de segunda a sexta-feira em horario agendado
- consultar os ativos monitorados
- buscar noticias relacionadas
- enviar os dados para a OpenAI gerar uma leitura executiva
- montar um email em HTML com painel comparativo e resumo
- disparar esse email automaticamente

Ativos monitorados atualmente:

- `PETR4`
- `TAEE11`
- `VALE3`
- `ITSA4`
- `BBAS3`
- `BBSE3`
- `MXRF11`
- `XPML11`
- `GARE11`
- `VGHF11`
- `VISC11`
- `BITCOIN`

## Arquitetura do workflow

Fluxo conceitual:

```text
Schedule Trigger
  -> Set Config
    -> Expandir Ativos -> HTTP Acoes BR
    -> HTTP Bitcoin
    -> HTTP Noticias BR
    -> HTTP Noticias BTC
  -> Merge
  -> Preparar Prompt
  -> OpenAI Analise
  -> Formatar Email
  -> Send an Email
```

O que cada parte faz:

- `Schedule Trigger`: dispara o workflow em dias uteis
- `Set Config`: concentra email de destino, ativos, queries de noticias e idioma
- `Expandir Ativos`: transforma a lista de acoes/FIIs em itens individuais
- `HTTP Acoes BR`: consulta a `brapi` ativo por ativo
- `HTTP Bitcoin`: consulta o `CoinGecko`
- `HTTP Noticias BR`: consulta noticias macro de acoes/FIIs
- `HTTP Noticias BTC`: consulta noticias de `Bitcoin`
- `Merge`: sincroniza os resultados antes da IA
- `Preparar Prompt`: organiza mercado + noticias + instrucoes para a OpenAI
- `OpenAI Analise`: gera a analise textual
- `Formatar Email`: converte a resposta em email HTML
- `Send an Email`: envia o relatorio

## Fontes de dados

- `brapi`: precos e informacoes de acoes/FIIs
- `CoinGecko`: preco e variacao de `Bitcoin`
- `GNews`: noticias recentes
- `OpenAI API`: geracao da analise

## Requisitos

- Docker Desktop
- Conta/API key da `OpenAI`
- Conta/API key da `brapi`
- Conta/API key da `GNews`
- Credencial SMTP valida para envio de email

## Rodando localmente com Docker

1. Copie o arquivo de exemplo:

```bash
cp .env.example .env
```

2. Preencha o `.env` com:

- `N8N_ENCRYPTION_KEY`
- `N8N_BASIC_AUTH_USER`
- `N8N_BASIC_AUTH_PASSWORD`
- `BRAPI_API_KEY`
- `GNEWS_API_KEY`
- `OPENAI_API_KEY`

3. Suba o `n8n`:

```bash
docker compose up -d
```

4. Abra o painel:

- [http://localhost:5678](http://localhost:5678)

5. Entre com o usuario e senha definidos no `.env`.

## Como importar e usar no n8n

1. No painel do `n8n`, clique em `Workflows`
2. Escolha `Import from file`
3. Selecione [n8n-workflow-analise-ativos.json](/Users/jacquesjacob/Documents/agente-investimentos/n8n-workflow-analise-ativos.json)
4. Revise os nos `Set Config`, `HTTP Acoes BR`, `HTTP Noticias BR`, `HTTP Noticias BTC`, `OpenAI Analise` e `Send an Email`
5. Configure a credencial SMTP no no `Send an Email`
6. Rode um teste manual
7. Clique em `Publish` para colocar o workflow em producao

## Configuracao importante no workflow

### 1. Set Config

Esse no concentra os parametros de negocio:

- email de destino
- lista de ativos
- query de noticias Brasil
- query de noticias de `Bitcoin`
- idioma/pais das noticias

Exemplo de bloco atual:

```json
{
  "email": "jacques@jacob.net.br",
  "stocks": [
    "PETR4",
    "TAEE11",
    "VALE3",
    "ITSA4",
    "BBAS3",
    "BBSE3",
    "MXRF11",
    "XPML11",
    "GARE11",
    "VGHF11",
    "VISC11"
  ],
  "cryptoId": "bitcoin",
  "newsQueryBrasil": "Petrobras OR Vale OR Itausa OR \"Banco do Brasil\" OR fii",
  "newsQueryBitcoin": "bitcoin OR btc",
  "gnewsLanguage": "pt",
  "gnewsCountry": "br"
}
```

### 2. brapi

O plano usado na `brapi` permitia apenas `1 ativo por requisicao`, por isso o workflow usa `Expandir Ativos` antes do no `HTTP Acoes BR`.

Isso evita erro do tipo:

- `Seu plano permite no maximo 1 ativo(s) por requisicao`

### 3. GNews

A `GNews` tem dois cuidados importantes:

- limite de taxa (`429`)
- limite de tamanho da query (`200 caracteres`)

Por isso as queries devem ser curtas e mais macro. Em vez de listar todos os tickers, a estrategia atual usa termos amplos como:

- `Petrobras`
- `Vale`
- `Itausa`
- `Banco do Brasil`
- `fii`

### 4. OpenAI

O no `OpenAI Analise` envia o prompt para a API `Responses` da OpenAI. O workflow foi ajustado para gerar:

- resumo executivo
- tabela comparativa curta com semaforo de risco
- conclusao por ativo
- recomendacao de `comprar aos poucos`, `segurar` ou `evitar aumentar posicao`

### 5. Email HTML

O no `Formatar Email` foi ajustado para:

- montar um cabecalho executivo
- renderizar um painel de mercado
- converter tabelas markdown da IA em tabela HTML real
- gerar um fallback `text/plain` mais limpo

## Como testar

1. Salve o workflow
2. Execute manualmente
3. Valide se os nos de mercado, noticias, IA e email concluem com sucesso
4. Confira o email recebido

Se o email chegar, mas com texto mal formatado:

- revise o no `Formatar Email`
- confirme que o no `Send an Email` usa `HTML`
- confirme que o campo `Subject` usa expressao e nao texto literal

## Como publicar em producao

No `n8n` atual, para o agendamento funcionar, o workflow precisa estar `Published`.

Passos:

1. teste manualmente
2. clique em `Publish`
3. confirme a publicacao
4. aguarde o proximo horario do `Schedule Trigger`

Observacao:

- em ambiente local, o agendamento so funciona se o notebook estiver ligado, sem dormir, com Docker rodando e o container `n8n-local` ativo

## Problemas comuns

### GNews retorna `429`

Causa:

- excesso de requisicoes

Mitigacoes:

- usar menos nos paralelos
- reduzir `max`
- simplificar query
- ativar `Retry on Fail`
- ativar `Never Error` quando fizer sentido

### GNews retorna `The query is too long`

Causa:

- query maior que `200 caracteres`

Mitigacao:

- usar palavras-chave macro em vez de todos os tickers

### brapi retorna erro por quantidade de ativos

Causa:

- o plano aceita poucos ativos por request

Mitigacao:

- usar `Expandir Ativos` e consultar um ticker por vez

### Workflow nao roda no horario

Checklist:

- workflow foi `Published`
- `Schedule Trigger` esta correto
- timezone esta certa
- container `n8n` esta rodando
- notebook nao entrou em repouso

## Comandos uteis

Subir:

```bash
docker compose up -d
```

Ver logs:

```bash
docker compose logs -f
```

Ver status:

```bash
docker compose ps
```

Parar:

```bash
docker compose down
```

## Seguranca

Arquivos sensiveis nao devem ir para o Git:

- `.env`
- `n8n_data/`

O projeto ja inclui [`.gitignore`](/Users/jacquesjacob/Documents/agente-investimentos/.gitignore) para proteger isso.

## Repositorio

Este projeto foi publicado no GitHub privado:

- [agente-investimentos](https://github.com/JacquesJacob/agente-investimentos)

## Aviso

Este workflow gera conteudo informativo. Ele nao substitui recomendacao personalizada de investimento, consultoria, suitability ou gestao profissional de carteira.
