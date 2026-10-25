# Desafio Criativo: Planejando Automações com N8N Usando Apenas Bons Prompts

Desafio do bootcamp **Santander 2026 - Automação com N8N** (DIO).

O objetivo era planejar uma automação no N8N a partir de um prompt claro e estruturado, construído passo a passo. Aqui estão as etapas, o prompt final e o teste dele com uma IA.

---

## Passo 1: Definir a automação

**Processo:** registrar automaticamente leads recebidos por formulário.

**Público ou responsável:** equipe comercial, que hoje copia os dados dos leads manualmente e corre o risco de perder contatos.

**Resultado esperado:** assim que um lead preencher o formulário, os dados são salvos em uma planilha e o time de vendas recebe uma notificação com as informações do novo lead.

---

## Passo 2: Ferramentas, fluxo e regras

**Ferramentas:** Google Forms, Google Sheets e Gmail.

**Fluxo:**
1. Receber uma nova resposta do formulário.
2. Validar o e-mail informado.
3. Registrar os dados na planilha de leads.
4. Avisar o time de vendas por e-mail.
5. Enviar e-mail de confirmação ao lead.

**Regra importante:** ignorar registros sem e-mail válido.

---

## Passo 3: Prompt final

```text
Atue como um especialista em N8N com experiência em automações comerciais e em integrações com Google Workspace.

CONTEXTO
Uma equipe comercial recebe leads por um Google Forms e hoje copia os dados manualmente para uma planilha, o que gera atraso e perda de contatos. Quero automatizar esse processo no N8N.

OBJETIVO
Planejar um workflow que registre cada novo lead em uma planilha organizada e avise o time de vendas na hora.

FERRAMENTAS
Google Forms, Google Sheets e Gmail.

DADOS DO LEAD
Nome, e-mail, telefone, interesse e data/hora do envio.

FLUXO DESEJADO
1. Gatilho: detectar uma nova resposta do Google Forms. Lembre que o N8N não tem gatilho nativo para o Forms, então use o gatilho de nova linha na planilha de respostas vinculada ao formulário (ou um webhook, se for mais adequado) e justifique a escolha.
2. Validação: verificar se o e-mail é válido (não vazio e com formato correto).
3. Registro: se for válido, adicionar uma nova linha em uma planilha de LEADS separada da planilha de respostas do Forms, para manter a base organizada.
4. Notificação interna: enviar um e-mail pelo Gmail para o time de vendas com os dados do novo lead.
5. Confirmação: enviar um e-mail pelo Gmail para o lead, agradecendo o contato.

REGRAS
- Se o e-mail for inválido ou estiver vazio, não salvar na planilha e não enviar nenhum e-mail.
- Nesse caso, o fluxo deve terminar de forma limpa, sem gerar erro.
- Evitar duplicidade: se o mesmo e-mail já existir na planilha de leads, não registrar de novo.

FORMATO DA RESPOSTA
Responda em português, em 4 partes:
1. Visão geral do workflow em 2 ou 3 frases.
2. Tabela com os nós do N8N na ordem de execução: nome do nó, função e principais configurações.
3. Lógica de funcionamento, explicando as ramificações (caminho válido e caminho inválido) e onde fica a validação, incluindo a expressão ou regex sugerida.
4. Pontos de atenção: erros comuns, credenciais necessárias (OAuth do Google) e como testar o workflow.

Não escreva código além das expressões do N8N necessárias.
```

---

## Teste do prompt

Resultado obtido ao rodar o prompt acima em uma IA:

### 1. Visão geral

O workflow é disparado quando uma nova resposta chega na planilha vinculada ao Google Forms. Ele valida o e-mail, confere se o lead já existe na planilha de leads e, se estiver tudo certo, registra a linha, avisa o time de vendas e confirma o recebimento para o lead. Se o e-mail for inválido ou duplicado, o fluxo termina sem erro e sem enviar nada.

### 2. Nós do N8N (ordem de execução)

| # | Nó | Função | Configuração principal |
|---|----|--------|------------------------|
| 1 | Google Sheets Trigger | Gatilho: detecta nova resposta | Evento "Row Added", planilha de respostas do Forms, verificação a cada 1 minuto |
| 2 | Edit Fields (Set) | Padroniza os campos | Cria nome, e-mail, telefone, interesse e data/hora, aplicando trim e minúsculas no e-mail |
| 3 | IF (E-mail válido?) | Validação | Condição com regex no campo e-mail; saída true segue o fluxo, saída false vai para o nó final |
| 4 | Google Sheets (Get Row(s)) | Busca duplicidade | Planilha de leads, filtro pela coluna e-mail igual ao e-mail recebido |
| 5 | IF (Já existe?) | Decide se registra | Se a busca retornou algum item, vai para o nó final; senão, segue |
| 6 | Google Sheets (Append Row) | Registra o lead | Planilha de leads, mapeando as 5 colunas |
| 7 | Gmail (Send a Message) | Avisa o time de vendas | Destinatário: e-mail do time; assunto "Novo lead: {{nome}}"; corpo com os dados |
| 8 | Gmail (Send a Message) | Confirma para o lead | Destinatário: e-mail do lead; mensagem de agradecimento |
| 9 | No Operation, do nothing | Fim limpo | Recebe os caminhos inválido e duplicado, sem gerar erro |

### 3. Lógica de funcionamento

O gatilho verifica a planilha de respostas e entrega cada nova linha ao workflow. O nó Edit Fields organiza os dados e limpa espaços do e-mail. No primeiro IF, a validação usa a expressão `{{ $json.email }}` com o operador "matches regex" e o padrão `^[^\s@]+@[^\s@]+\.[^\s@]+$`. Se não passar, o item vai direto para o No Operation e o fluxo acaba.

Se passar, o workflow consulta a planilha de leads para ver se aquele e-mail já foi registrado. Existindo, também encerra no No Operation. Não existindo, o Append Row grava o lead, o primeiro Gmail avisa a equipe comercial e o segundo Gmail envia a confirmação ao lead. Os dois e-mails podem rodar em sequência ou em paralelo.

Usar uma planilha de leads separada da planilha de respostas evita misturar dados brutos com dados já validados.

### 4. Pontos de atenção

- Configurar as credenciais OAuth2 do Google (Sheets e Gmail) no N8N e autorizar os escopos corretos.
- O Google Sheets Trigger usa polling, então pode haver pequeno atraso; para tempo real, usar webhook com Apps Script.
- Cuidado com nomes de colunas: se o cabeçalho da planilha mudar, o mapeamento quebra.
- O Gmail tem limite diário de envios; para volumes altos, considerar outro serviço de e-mail.
- Testar com três cenários: e-mail válido, e-mail inválido e e-mail repetido.

---

## O que aprendi

Um bom prompt separa contexto, objetivo, regras e formato de resposta. Foi incluir o gatilho explícito, a distinção entre os dois e-mails, o tratamento do caso inválido e o formato da resposta que fez a IA entregar um plano completo de primeira, sem precisar adivinhar nada.
