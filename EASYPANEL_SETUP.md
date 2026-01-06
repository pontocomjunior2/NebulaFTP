# 🚀 Guia de Instalação: NebulaFTP no EasyPanel

Este guia explica como configurar o NebulaFTP no EasyPanel usando Dockerfile diretamente.

---

## 📋 Pré-requisitos

1. **EasyPanel** instalado e funcionando
2. **API ID e API HASH**: Obtenha em [my.telegram.org](https://my.telegram.org)
3. **BOT TOKEN**: Crie um bot com [@BotFather](https://t.me/BotFather)
4. **CHAT ID**: Crie um Canal Privado, adicione o Bot como Admin e pegue o ID no [@userinfobot](https://t.me/userinfobot)
5. **MongoDB**: Configure um banco MongoDB (pode ser outro container no EasyPanel ou MongoDB Atlas)

---

## 🔧 Passo 1: Criar Aplicação no EasyPanel

1. Acesse o EasyPanel
2. Clique em **"New Application"** ou **"Criar Aplicação"**
3. Selecione **"Dockerfile"** como tipo de aplicação
4. Configure:
   - **Nome**: `nebula-ftp` (ou o nome que preferir)
   - **Repositório**: URL do seu repositório Git (ex: `https://github.com/pontocomjunior2/NebulaFTP.git`)
   - **Branch**: `master` (ou a branch que você está usando)
   - **Dockerfile Path**: `Dockerfile` (deixe como está se estiver na raiz)

---

## 🔌 Passo 2: Configurar Portas no EasyPanel

No EasyPanel, você precisa expor as portas **individualmente** (não há suporte a ranges).

### Porta de Controle FTP

1. Clique em **"Adicionar Porta"**
2. Configure:
   - **Protocolo**: `tcp`
   - **Publicado**: `2121`
   - **Alvo**: `2121`

### Portas Passivas (Múltiplas Portas Individuais)

Como o EasyPanel não suporta ranges, você precisa adicionar **cada porta individualmente**.

**Recomendação**: Use um range pequeno para não ser muito trabalhoso (ex: `60000-60010` = 11 portas).

Para cada porta do range `60000-60010`, adicione uma porta:

1. Clique em **"Adicionar Porta"**
2. Configure:
   - **Protocolo**: `tcp`
   - **Publicado**: `60000` (primeira porta)
   - **Alvo**: `60000`
3. Repita para cada porta: `60001`, `60002`, `60003`, ... até `60010`

**Total**: Você terá 11 portas passivas (60000-60010) + 1 porta de controle (2121) = **12 portas no total**.

**Dica**: Se precisar de mais portas simultâneas, adicione mais portas ao range (ex: `60000-60020` = 21 portas).

---

## ⚙️ Passo 3: Configurar Variáveis de Ambiente no EasyPanel

No painel do EasyPanel, adicione as seguintes variáveis de ambiente:

### Variáveis Obrigatórias

```env
# Telegram
API_ID=12345678
API_HASH=abc123def456789abcdef123456789ab
BOT_TOKEN=1234567890:AABBccDDeeFFggHHiiJJkkLLmmNN
CHAT_ID=-1001234567890

# MongoDB
MONGODB=mongodb://mongo:27017
# OU se usar MongoDB Atlas:
# MONGODB=mongodb+srv://usuario:senha@cluster.mongodb.net/ftp

# Servidor FTP
HOST=0.0.0.0
PORT=2121
```

### Variáveis para Modo Passivo (IMPORTANTE para EasyPanel)

```env
# Range de portas passivas (deve corresponder às portas expostas no Passo 2)
# Use um range pequeno (ex: 60000-60010) para não adicionar muitas portas no EasyPanel
FTP_PASV_PORTS=60000-60010

# IP público do servidor (CRÍTICO - use o IP público do EasyPanel)
# No seu caso: 93.127.141.215
FTP_MASQUERADE_ADDRESS=93.127.141.215
```

**Nota sobre o IP público:**
- No EasyPanel, o IP público geralmente é o IP do próprio servidor EasyPanel
- No seu caso, use: `93.127.141.215`
- **IMPORTANTE**: Use o IP público do EasyPanel, não o IP interno do container!

### Variáveis Opcionais (Performance)

```env
MAX_WORKERS=4
CHUNK_SIZE_MB=64
MAX_RETRIES=5
MAX_STAGING_AGE=3600
LOG_LEVEL=INFO
```

---

## 🔍 Passo 4: Verificar Configuração

Após configurar tudo, verifique os logs do container no EasyPanel. Você deve ver:

```
🔓 Portas Passivas definidas: 60000-60100
📡 Masquerade Address definido: SEU_IP_PUBLICO
🚀 Nebula FTP (MonoBot) Rodando na porta 2121
```

---

## 🐛 Troubleshooting

### Erro: "425 bad sequence (no data connection)"

**Causa**: O servidor está retornando o IP interno do container na resposta PASV.

**Solução**:
1. Verifique se `FTP_MASQUERADE_ADDRESS` está configurado com o IP público correto
2. Verifique se as portas passivas estão expostas no EasyPanel
3. Verifique se o range em `FTP_PASV_PORTS` corresponde às portas expostas

### Erro: "no available ports in range"

**Causa**: As portas do range não estão disponíveis ou não estão expostas.

**Solução**:
1. Verifique se o range de portas está exposto no EasyPanel
2. Tente usar um range diferente (ex: `60000-60010` para menos portas)
3. Verifique se outras aplicações não estão usando essas portas

### Cliente FTP não consegue conectar

**Solução**:
1. Verifique se o firewall do servidor permite as portas:
   ```bash
   # Ubuntu/Debian
   sudo ufw allow 2121/tcp
   sudo ufw allow 60000:60100/tcp
   ```
2. Verifique se o `FTP_MASQUERADE_ADDRESS` está correto
3. Teste a conexão com um cliente FTP (FileZilla, WinSCP, etc.)

---

## 📝 Exemplo Completo de Variáveis de Ambiente

Aqui está um exemplo completo de todas as variáveis que você deve configurar no EasyPanel:

```env
# ============= TELEGRAM =============
API_ID=12345678
API_HASH=abc123def456789abcdef123456789ab
BOT_TOKEN=1234567890:AABBccDDeeFFggHHiiJJkkLLmmNN
CHAT_ID=-1001234567890

# ============= MONGODB =============
# Opção 1: MongoDB em outro container no EasyPanel
MONGODB=mongodb://mongo:27017

# Opção 2: MongoDB Atlas (Cloud)
# MONGODB=mongodb+srv://usuario:senha@cluster.mongodb.net/ftp

# ============= SERVIDOR FTP =============
HOST=0.0.0.0
PORT=2121

# ============= CONFIGURAÇÕES PASSIVAS (OBRIGATÓRIO PARA EASYPANEL) =============
# Use um range pequeno (ex: 60000-60010) para não adicionar muitas portas no EasyPanel
FTP_PASV_PORTS=60000-60010
FTP_MASQUERADE_ADDRESS=93.127.141.215  # IP público do EasyPanel

# ============= PERFORMANCE =============
MAX_WORKERS=4
CHUNK_SIZE_MB=64
MAX_RETRIES=5
MAX_STAGING_AGE=3600

# ============= LOGGING =============
LOG_LEVEL=INFO
```

---

## ✅ Checklist de Configuração

Antes de testar a conexão FTP, verifique:

- [ ] Todas as variáveis de ambiente estão configuradas no EasyPanel
- [ ] `FTP_MASQUERADE_ADDRESS` está com o IP público correto do servidor
- [ ] `FTP_PASV_PORTS` corresponde às portas individuais expostas no EasyPanel
- [ ] Porta 2121 está exposta no EasyPanel
- [ ] Todas as portas passivas (60000-60010) foram adicionadas individualmente no EasyPanel
- [ ] Firewall do servidor permite as portas
- [ ] MongoDB está acessível (se usar container separado, verifique a rede)
- [ ] Logs do container mostram as mensagens de configuração corretas

---

## 🔗 Próximos Passos

Após configurar tudo:

1. **Criar usuário FTP**: Acesse o container e execute:
   ```bash
   python accounts_manager.py
   ```

2. **Testar conexão**: Use um cliente FTP (FileZilla, WinSCP, etc.) para testar a conexão

3. **Monitorar logs**: Acompanhe os logs no EasyPanel para verificar se tudo está funcionando

---

## 📚 Referências

- [Documentação do EasyPanel](https://easypanel.io/docs)
- [Guia de Configuração do Telegram](docs/TELEGRAM_SETUP.md)
- [Solução de Problemas FTP Passivo](SOLUCAO_FTP_PASV.md)

