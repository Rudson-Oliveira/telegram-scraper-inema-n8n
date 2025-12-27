# 📱 Telegram Scraper INEMA - N8N Workflow

> **Workflow automatizado para scraping, classificação IA e análise de canais Telegram do INEMA**

[![N8N](https://img.shields.io/badge/N8N-Workflow-EA4B71?logo=n8n&logoColor=white)](https://n8n.io/)
[![Telegram](https://img.shields.io/badge/Telegram-Bot-26A5E4?logo=telegram&logoColor=white)](https://telegram.org/)
[![Supabase](https://img.shields.io/badge/Supabase-Database-3ECF8E?logo=supabase&logoColor=white)](https://supabase.com/)
[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 📋 Descrição

Workflow completo em N8N para automação de scraping de mensagens de canais Telegram do INEMA, incluindo:

- 🤖 **Classificação via IA** - Categorização automática de mensagens
- 😊 **Análise de Sentimento** - Detecção de sentimentos em português
- 📊 **Extração de Conteúdo** - Identificação de prompts, ferramentas e recursos
- 💾 **Armazenamento Supabase** - Persistência com UPSERT automático
- 📧 **Notificações Email** - Alertas via Outlook
- 📱 **Notificações Telegram** - Mensagens instantâneas via bot

## 🏗️ Arquitetura do Workflow

```
┌─────────────────┐
│ Schedule Trigger│
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  HTTP Request   │ ← Busca mensagens da API Manus
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│Extract Messages │ ← Extrai dados JSON
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│Split In Batches │ ← Processa em lotes
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│Classificador IA │ ← Categorização automática
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│Análise Sentiment│ ← Score de sentimento
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│Extrator Conteúdo│ ← Extrai prompts/tools
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  💾 Supabase    │ ← Salva com UPSERT
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│IF Verificar Erro│
└────┬────────┬───┘
     │        │
  ✅ │        │ ❌
     │        │
     ▼        ▼
┌─────┐  ┌────────┐
│Email│  │Webhook │
│Tele │  │  Erro  │
│gram │  └────────┘
└─────┘
```

## ⚙️ Configuração

### Pré-requisitos

- ✅ Conta N8N (cloud ou self-hosted)
- ✅ API Manus para scraping Telegram
- ✅ Banco Supabase configurado
- ✅ Bot Telegram criado
- ✅ Conta Microsoft Outlook

### Variáveis de Ambiente

```env
# Manus API
MANUS_API_URL=https://tele-scrap-fgfuwhsp.manus.space/api/v1/messages
MANUS_API_KEY=tgs_...

# Supabase
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_ROLE=eyJhbG...

# Telegram Bot
TELEGRAM_BOT_TOKEN=8517983740:AAGTKHj9odaggGlo2EHBuXfrwYSazotCIK8
TELEGRAM_CHAT_ID=1070395107

# Email
OUTLOOK_EMAIL=rud.pa@otmail.com
```

### Credenciais N8N

1. **HTTP Request (Manus)**
   - Header: `X-Api-Key`
   - Value: Token Manus

2. **Telegram Bot**
   - Access Token: Token do @BotFather
   - Chat ID: ID do usuário/grupo

3. **Microsoft Outlook**
   - OAuth2 Authentication

4. **Supabase**
   - Hardcoded no node de Code
   - URL + Service Role Key

## 🚀 Como Usar

### 1. Importar Workflow

```bash
# Baixar o workflow JSON
curl -O https://github.com/Rudson-Oliveira/telegram-scraper-inema-n8n/raw/main/workflow.json

# Importar no N8N via Interface
# Menu → Import from File → workflow.json
```

### 2. Configurar Credenciais

- Configure todas as credenciais nos nodes
- Teste cada conexão individualmente
- Valide o acesso ao Supabase

### 3. Criar Bot Telegram

```bash
# No Telegram, busque @BotFather
/newbot
# Nome: INEMA Notifications
# Username: inema_notifications_bot
# Copie o token fornecido
```

### 4. Obter Chat ID

```bash
# Busque @userinfobot no Telegram
/start
# Copie o ID fornecido
```

### 5. Ativar Workflow

- Clique em "Activate" no N8N
- Configure o Schedule Trigger
- Monitore os logs de execução

## 📊 Estrutura do Banco (Supabase)

### Tabela: `telegram_messages`

```sql
CREATE TABLE telegram_messages (
  id TEXT PRIMARY KEY,
  chat_id BIGINT,
  message_date TIMESTAMP,
  text TEXT,
  
  -- Classificação
  category TEXT,
  classification_confidence NUMERIC,
  classification_reasoning TEXT,
  classified_at TIMESTAMP,
  
  -- Sentimento
  sentiment TEXT,
  sentiment_score NUMERIC,
  sentiment_reasoning TEXT,
  
  -- Conteúdo Extraído
  extracted_prompts JSONB,
  extracted_tools JSONB,
  extracted_resources JSONB,
  
  -- Metadados
  scraped_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

## 📱 Notificações

### Email (Outlook)

**Assunto:** ⚠️ Novas Mensagens INEMA - Telegram Scraper

**Corpo:**
```
Olá Rudson,

Novas mensagens foram detectadas nos canais INEMA:

🔹 Total processado: {count}
✅ Dados salvos no Supabase
⏰ Processado em: {timestamp}
```

### Telegram Bot

**Mensagem:**
```
🔔 Novas Mensagens INEMA - Telegram Scraper

Novas mensagens foram detectadas e processadas com sucesso nos canais do INEMA.

✅ Dados salvos no Supabase
⏰ Processado em: 27/12/2025 10:18:49
```

## 🧪 Testes

### Testar Node Individual

1. Selecione o node no workflow
2. Clique em "Execute Node" 
3. Verifique o output no painel direito

### Testar Workflow Completo

```bash
# Via N8N UI
Clique em "Execute Workflow" → Selecione trigger

# Via API (webhook)
curl -X POST https://your-n8n-instance.app.n8n.cloud/webhook/...
```

## 📈 Monitoramento

### Executions Tab

- ✅ **Sucesso**: Verde com detalhes
- ❌ **Erro**: Vermelho com stack trace
- ⏸️ **Aguardando**: Amarelo

### Logs

Acesse os logs via:
- N8N UI → Logs panel (bottom)
- Console do navegador (F12)
- Supabase dashboard → Logs

## 🔧 Troubleshooting

### Erro: "Supabase connection failed"

```bash
# Verifique credenciais
- URL correto?
- Service Role Key válido?
- Tabela existe?
```

### Erro: "Telegram bot not responding"

```bash
# Verifique token e Chat ID
- Token válido?
- Chat ID correto?
- Bot iniciado com /start?
```

### Erro: "No messages found"

```bash
# Verifique API Manus
- Endpoint acessível?
- API Key válido?
- Canais configurados?
```

## 📝 Changelog

### v1.0.0 (27/12/2025)
- ✅ Workflow inicial criado
- ✅ Integração Manus API
- ✅ Bot Telegram configurado (@inema_notifications_bot)
- ✅ Notificações Email implementadas
- ✅ Supabase integrado
- ✅ Testes realizados com sucesso

## 🤝 Contribuindo

Contribuições são bem-vindas! Para contribuir:

1. Fork o projeto
2. Crie uma branch (`git checkout -b feature/NovaFeature`)
3. Commit suas mudanças (`git commit -m 'Add: Nova Feature'`)
4. Push para a branch (`git push origin feature/NovaFeature`)
5. Abra um Pull Request

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

## 👤 Autor

**Rudson Oliveira**

- GitHub: [@Rudson-Oliveira](https://github.com/Rudson-Oliveira)
- Email: rud.pa@otmail.com
- Telegram: Chat ID 1070395107

## 🙏 Agradecimentos

- [N8N](https://n8n.io/) - Plataforma de automação
- [Supabase](https://supabase.com/) - Backend as a Service
- [Telegram](https://telegram.org/) - Messaging platform
- [Manus](https://manus.space/) - Telegram Scraping API

---

**Status:** ✅ Operacional em Produção | **Última atualização:** 27/12/2025 10:00 AM
