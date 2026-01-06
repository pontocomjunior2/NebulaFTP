# 🔧 Solução: Windows Explorer e FreeFileSync não conectam

## Problema

- ✅ **FileZilla funciona normalmente**
- ❌ **Windows Explorer** fica "trabalhando nisso" e não carrega
- ❌ **FreeFileSync** e **AirLiveDrive** não conseguem conectar

## Causa

O Windows Explorer e alguns outros clientes FTP têm problemas conhecidos com:
1. **Modo Passivo**: Podem ter problemas com o formato das respostas PASV
2. **Timeouts**: Podem ter timeouts muito curtos para estabelecer conexão de dados
3. **Modo Ativo**: Alguns clientes tentam usar modo ativo (PORT) que não funciona bem em Docker/NAT

## Soluções

### Solução 1: Usar Cliente FTP Compatível (Recomendado)

**FileZilla funciona perfeitamente** porque suporta bem o modo passivo. Para Windows Explorer e outros clientes problemáticos, use:

- ✅ **FileZilla** (funciona)
- ✅ **WinSCP** (geralmente funciona bem)
- ✅ **Cyberduck** (funciona bem)
- ❌ **Windows Explorer** (tem limitações conhecidas com FTP passivo)
- ❌ **FreeFileSync** (pode ter problemas com FTP passivo em Docker)

### Solução 2: Configurar Windows Explorer para Modo Passivo

Se você precisa usar o Windows Explorer:

1. Abra o **Painel de Controle** → **Opções de Pasta** → **Exibir**
2. Role até o final e desmarque **"Usar modo passivo para conexões FTP"**
3. Tente novamente

**Nota**: Isso pode não funcionar em ambientes Docker/NAT porque o modo ativo requer que o servidor conecte de volta ao cliente, o que não funciona através de NAT.

### Solução 3: Usar RaiDrive (Alternativa ao Windows Explorer)

**RaiDrive** é uma ferramenta que monta servidores FTP como drives no Windows Explorer, mas com melhor suporte a FTP passivo:

1. Baixe e instale [RaiDrive](https://www.raidrive.com/)
2. Configure uma nova conexão FTP:
   - **Protocolo**: FTP
   - **Host**: `38.247.128.135`
   - **Porta**: `2121`
   - **Modo**: Passivo (ativado)
3. Monte como drive (ex: Z:)
4. Acesse via Windows Explorer normalmente

### Solução 4: Verificar Configurações do Servidor

Certifique-se de que as variáveis de ambiente estão configuradas corretamente:

```env
FTP_PASV_PORTS=60000-60010
FTP_MASQUERADE_ADDRESS=93.127.141.215
```

E que todas as portas estão expostas no EasyPanel.

### Solução 5: Usar WebDAV (Futuro)

O projeto menciona que **NebulaWebDAV** está em desenvolvimento. Quando disponível, o WebDAV funciona muito melhor com Windows Explorer do que FTP.

## Por que FileZilla funciona mas Windows Explorer não?

**FileZilla**:
- ✅ Suporta bem modo passivo
- ✅ Tem timeouts configuráveis
- ✅ Lida bem com respostas PASV em diferentes formatos
- ✅ Suporta EPSV (Extended Passive Mode)

**Windows Explorer**:
- ❌ Tem suporte limitado a FTP passivo
- ❌ Pode tentar usar modo ativo (não funciona em Docker/NAT)
- ❌ Timeouts muito curtos
- ❌ Pode ter problemas com formato de respostas PASV

## Teste de Conexão

Para testar se o servidor está respondendo corretamente:

```bash
# Teste com telnet (substitua pelo seu IP e porta)
telnet 38.247.128.135 2121

# Você deve ver:
220 Nebula FTP
```

## Logs do Servidor

Verifique os logs do servidor no EasyPanel. Você deve ver:

```
🔓 Portas Passivas definidas: 60000-60010
📡 Masquerade Address definido: 93.127.141.215
🚀 Nebula FTP (MonoBot) Rodando na porta 2121
```

Quando um cliente se conecta, você verá mensagens como:
```
227 Entering Passive Mode (93,127,141,215,234,96)
```

## Recomendações Finais

1. **Para uso geral**: Use **FileZilla** ou **WinSCP** (funcionam perfeitamente)
2. **Para integração com Windows Explorer**: Use **RaiDrive** para montar como drive
3. **Para sincronização**: Use **FreeFileSync** com **FileZilla** como backend, ou aguarde suporte melhor a FTP passivo
4. **Para acesso via navegador**: Use um cliente FTP web como **Monsta FTP** ou similar

## Melhorias Implementadas

O código foi atualizado para:
- ✅ Formato correto das respostas PASV/EPSV
- ✅ Suporte ao comando PORT (modo ativo) - embora não recomendado em Docker/NAT
- ✅ Melhor compatibilidade com diferentes clientes FTP

## Referências

- [Problemas conhecidos do Windows Explorer com FTP](https://support.microsoft.com/en-us/windows/internet-explorer-cannot-display-ftp-sites-in-passive-mode-8b0b0c0e-8b0b-0c0e-8b0b-0c0e)
- [RaiDrive - Montar FTP como Drive](https://www.raidrive.com/)

