# n8n local via Docker

Arquivos:

- [docker-compose.yml](/Users/jacquesjacob/Documents/agente-investimentos/docker-compose.yml)
- [.env.example](/Users/jacquesjacob/Documents/agente-investimentos/.env.example)
- [n8n-workflow-analise-ativos.json](/Users/jacquesjacob/Documents/agente-investimentos/n8n-workflow-analise-ativos.json)

## O que este setup faz

- Sobe o `n8n` localmente em `http://localhost:5678`
- Mantem os dados persistidos em `./n8n_data`
- Protege o acesso com usuario e senha
- Deixa prontas as variaveis para `BRAPI`, `GNews` e `OpenAI`

## Passo 1: criar o arquivo .env

Copie o modelo:

```bash
cp .env.example .env
```

Depois edite o arquivo `.env` e preencha:

- `N8N_ENCRYPTION_KEY`
- `N8N_BASIC_AUTH_USER`
- `N8N_BASIC_AUTH_PASSWORD`
- `BRAPI_API_KEY`
- `GNEWS_API_KEY`
- `OPENAI_API_KEY`

## Passo 2: subir o n8n

```bash
docker compose up -d
```

## Passo 3: abrir o painel

Abra:

- [http://localhost:5678](http://localhost:5678)

Entre com o usuario e a senha do `.env`.

## Passo 4: importar o workflow

1. No n8n, clique em `Workflows`.
2. Escolha `Import from file`.
3. Selecione [n8n-workflow-analise-ativos.json](/Users/jacquesjacob/Documents/agente-investimentos/n8n-workflow-analise-ativos.json).

## Passo 5: ajustar o envio de email

No no `Enviar Email`:

- troque `fromEmail`
- configure a credencial SMTP

## Comandos uteis

Subir:

```bash
docker compose up -d
```

Ver logs:

```bash
docker compose logs -f
```

Parar:

```bash
docker compose down
```
