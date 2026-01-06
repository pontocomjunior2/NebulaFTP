# 🔧 Solução: Erro 425 bad sequence (no data connection)

## Problema
O servidor FTP está retornando o IP interno do container (ex: `93.127.141.215`) na resposta PASV, fazendo com que o cliente FTP não consiga estabelecer a conexão de dados.

## Solução

### Para EasyPanel (Dockerfile direto)

Se você está usando EasyPanel com Dockerfile diretamente (não docker-compose):

### 1. Configure as Variáveis de Ambiente no EasyPanel

No painel do EasyPanel, adicione as seguintes variáveis de ambiente:

```env
# IP público do EasyPanel (no seu caso: 93.127.141.215)
FTP_MASQUERADE_ADDRESS=93.127.141.215

# Range de portas passivas (use um range pequeno para não adicionar muitas portas)
# Deve corresponder às portas individuais expostas no EasyPanel
FTP_PASV_PORTS=60000-60010
```

**Como descobrir seu IP público:**
- No servidor: `curl ifconfig.me` ou `curl ipinfo.io/ip`
- Ou use o IP público que você usa para acessar o servidor via SSH

### 2. Configure as Portas no EasyPanel

**IMPORTANTE**: No EasyPanel, você precisa adicionar **cada porta individualmente** (não há suporte a ranges).

#### Porta de Controle FTP
1. Clique em **"Adicionar Porta"**
2. Configure:
   - **Protocolo**: `tcp`
   - **Publicado**: `2121`
   - **Alvo**: `2121`

#### Portas Passivas (Adicionar Individualmente)

Para cada porta do range configurado em `FTP_PASV_PORTS`, adicione uma porta:

**Exemplo para range `60000-60010` (11 portas):**

1. Clique em **"Adicionar Porta"** → Configure: `tcp`, Publicado: `60000`, Alvo: `60000`
2. Clique em **"Adicionar Porta"** → Configure: `tcp`, Publicado: `60001`, Alvo: `60001`
3. Clique em **"Adicionar Porta"** → Configure: `tcp`, Publicado: `60002`, Alvo: `60002`
4. ... continue até `60010`

**Total**: 11 portas passivas (60000-60010) + 1 porta de controle (2121) = **12 portas no total**.

**Dica**: Use um range pequeno (ex: `60000-60010`) para não adicionar muitas portas. Se precisar de mais conexões simultâneas, aumente o range.

### 3. Reinicie a Aplicação no EasyPanel

No painel do EasyPanel, reinicie a aplicação para aplicar as novas variáveis de ambiente.

### 4. Verifique os Logs

No EasyPanel, acesse os logs da aplicação. Você deve ver:

Você deve ver mensagens como:
```
🔓 Portas Passivas definidas: 60000-60010
📡 Masquerade Address definido: 93.127.141.215
```

### 5. Teste a Conexão

Ao conectar via FileZilla ou outro cliente FTP, a resposta PASV deve mostrar o IP público configurado, não o IP interno do container.

## Exemplo Completo de .env

```env
# Telegram
API_ID=12345678
API_HASH=abc123def456...
BOT_TOKEN=1234567890:AABBcc...
CHAT_ID=-1001234567890

# MongoDB
MONGODB=mongodb://mongo:27017

# Servidor FTP
HOST=0.0.0.0
PORT=2121

# Configurações para Docker sem network_mode: host (EasyPanel)
FTP_PASV_PORTS=60000-60010  # Use range pequeno para não adicionar muitas portas
FTP_MASQUERADE_ADDRESS=93.127.141.215  # IP público do EasyPanel

# Performance
MAX_WORKERS=4
CHUNK_SIZE_MB=64
MAX_RETRIES=5
MAX_STAGING_AGE=3600

# Logging
LOG_LEVEL=INFO
```

## Troubleshooting

### Se ainda não funcionar:

1. **Verifique se as portas estão abertas no firewall:**
   ```bash
   # Ubuntu/Debian
   sudo ufw allow 2121/tcp
   sudo ufw allow 60000:60100/tcp
   ```

2. **Verifique se o IP público está correto:**
   - O IP deve ser o IP público do servidor, não o IP interno do container
   - Use `curl ifconfig.me` para confirmar

3. **Verifique se todas as portas foram adicionadas:**
   - O `FTP_PASV_PORTS` nas variáveis de ambiente deve corresponder às portas individuais adicionadas no EasyPanel
   - Exemplo: Se `FTP_PASV_PORTS=60000-60010`, então você deve ter 11 portas individuais (60000, 60001, 60002, ..., 60010) no EasyPanel
   - Verifique se todas as portas do range foram adicionadas individualmente

4. **Verifique os logs no EasyPanel:**
   - Acesse os logs da aplicação no painel do EasyPanel
   - Procure por mensagens contendo "passiv" ou "masquerade"
   - Você deve ver: `🔓 Portas Passivas definidas: 60000-60010` e `📡 Masquerade Address definido: 93.127.141.215`

## 📖 Documentação Adicional

Para mais detalhes sobre configuração no EasyPanel, consulte:
- [Guia Completo EasyPanel](EASYPANEL_SETUP.md)

