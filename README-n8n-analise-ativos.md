# Workflow n8n: analise diaria de ativos

Arquivo principal:

- [n8n-workflow-analise-ativos.json](/Users/jacquesjacob/Documents/agente-investimentos/n8n-workflow-analise-ativos.json)

## O que este workflow faz

- Roda de segunda a sexta-feira as `08:00`
- Consulta `PETR4`, `VALE3`, `BBAS3`, `ITSA3`, `MXRF11` e `BITCOIN`
- Busca noticias recentes relacionadas aos ativos
- Pede para a OpenAI gerar uma analise em portugues
- Envia o resumo por email para `jacques@jacob.net.br`

## O que falta configurar no n8n

1. Importar o arquivo JSON do workflow.
2. Configurar as variaveis de ambiente:
   - `BRAPI_API_KEY`
   - `GNEWS_API_KEY`
   - `OPENAI_API_KEY`
3. Configurar a credencial SMTP usada no no `Enviar Email`.
4. Trocar o campo `fromEmail` do no `Enviar Email`.
5. Fazer um teste manual antes de ativar o workflow.

## Como importar

1. No n8n, clique em `Workflows`.
2. Escolha `Import from file`.
3. Selecione o arquivo `n8n-workflow-analise-ativos.json`.

## Observacoes

- O workflow foi deixado como `inactive` para evitar disparo antes das credenciais estarem prontas.
- Se preferir, o no `Enviar Email` pode ser trocado por `Gmail` em vez de SMTP.
- A classificacao `comprar aos poucos`, `segurar` e `evitar aumentar posicao` foi pensada como resumo informativo, nao como recomendacao financeira personalizada.
