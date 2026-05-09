# Instalando Node.js no Windows 11 e habilitando o Context7 MCP no Claude Code

> Tutorial passo a passo para configurar um ambiente de desenvolvimento com **Node.js** no Windows 11 e habilitar o **Context7 MCP** no **Claude Code** rodando pelo terminal do VS Code.
>
> O Context7 fornece documentação atualizada de bibliotecas (React, Next.js, FastAPI, etc.) diretamente para o Claude, evitando respostas com APIs desatualizadas ou inexistentes.

---

## Sumário

- [Pré-requisitos](#pré-requisitos)
- [Parte 1 — Instalando o Node.js no Windows 11](#parte-1--instalando-o-nodejs-no-windows-11)
  - [Método 1: Instalador oficial (recomendado para iniciantes)](#método-1-instalador-oficial-recomendado-para-iniciantes)
  - [Método 2: Winget (linha de comando)](#método-2-winget-linha-de-comando)
  - [Método 3: NVM for Windows (múltiplas versões)](#método-3-nvm-for-windows-múltiplas-versões)
  - [Verificando a instalação](#verificando-a-instalação)
- [Parte 2 — Habilitando o Context7 MCP no Claude Code](#parte-2--habilitando-o-context7-mcp-no-claude-code)
  - [Opção A: Servidor local via npx](#opção-a-servidor-local-via-npx)
  - [Opção B: Servidor remoto (SSE)](#opção-b-servidor-remoto-sse)
  - [Escopo da instalação](#escopo-da-instalação)
  - [Verificando se o MCP está conectado](#verificando-se-o-mcp-está-conectado)
- [Parte 3 — Como usar o Context7 no dia a dia](#parte-3--como-usar-o-context7-no-dia-a-dia)
- [Solução de problemas](#solução-de-problemas)
- [Referências](#referências)

---

## Pré-requisitos

- Windows 11 (também funciona no Windows 10)
- VS Code instalado
- Claude Code já instalado e funcionando no terminal
- Conexão com a internet

---

## Parte 1 — Instalando o Node.js no Windows 11

Existem três caminhos. Escolha **um** baseado no seu perfil:

| Método | Indicado para | Dificuldade |
|--------|---------------|-------------|
| Instalador oficial | Iniciantes / uso simples | ⭐ |
| Winget | Quem prefere linha de comando | ⭐⭐ |
| NVM for Windows | Devs que trabalham com múltiplas versões | ⭐⭐⭐ |

### Método 1: Instalador oficial (recomendado para iniciantes)

1. Acesse o site oficial: **https://nodejs.org**
2. Baixe a versão **LTS** (atualmente Node.js v24 LTS — versão estável de longo prazo).
3. Execute o arquivo `.msi` baixado.
4. Avance pelo instalador aceitando os termos de licença.
5. **Importante:** marque a opção **"Automatically install the necessary tools"**.
   - Isso instala automaticamente Chocolatey, Python e ferramentas de build necessárias para compilar pacotes npm que dependem de C/C++.
6. Clique em **Install** e autorize quando o Windows pedir permissão de administrador.
7. Após a instalação principal, um PowerShell pode abrir pedindo para apertar uma tecla — pressione qualquer tecla e aguarde (pode levar alguns minutos para baixar e configurar as ferramentas extras).
8. Clique em **Finish** ao terminar.

### Método 2: Winget (linha de comando)

O Winget é o gerenciador de pacotes nativo do Windows. Abra o PowerShell e execute:

```powershell
winget install OpenJS.NodeJS.LTS
```

Após a instalação, **feche e reabra o terminal** para que a variável `PATH` seja atualizada.

### Método 3: NVM for Windows (múltiplas versões)

Use este método se você trabalha com projetos que exigem versões diferentes do Node.

> ⚠️ **Atenção:** desinstale qualquer instalação prévia do Node antes de instalar o NVM, para evitar conflitos.

1. Baixe o instalador em: **https://github.com/coreybutler/nvm-windows/releases**
   - Procure o arquivo `nvm-setup.exe` na release mais recente.
2. Execute o instalador e siga os passos padrão.
3. Abra o **PowerShell como administrador** e instale a versão LTS:

```powershell
nvm install lts
nvm use lts
```

Para alternar entre versões depois:

```powershell
nvm list           # lista versões instaladas
nvm install 20.11.0  # instala uma versão específica
nvm use 20.11.0    # ativa essa versão
```

### Verificando a instalação

Independentemente do método escolhido, **abra um novo terminal** (PowerShell ou o terminal integrado do VS Code) e execute:

```bash
node --version
npm --version
```

Você deve ver as versões de cada um, por exemplo:

```
v24.0.0
10.9.0
```

Se aparecer "command not found" ou similar, reinicie o computador para garantir que o `PATH` foi atualizado.

---

## Parte 2 — Habilitando o Context7 MCP no Claude Code

O **Context7** é um servidor MCP (Model Context Protocol) desenvolvido pela Upstash que fornece documentação atualizada de bibliotecas em tempo real para o Claude.

### Opção A: Servidor local via npx

Este é o método mais comum. Roda o servidor localmente usando o `npx`. No terminal integrado do VS Code, execute:

```bash
claude mcp add context7 -- npx -y @upstash/context7-mcp@latest
```

> 💡 O `--` separa os argumentos do `claude mcp add` dos argumentos do comando que será executado.

### Opção B: Servidor remoto (SSE)

Se preferir não rodar o servidor localmente, conecte-se ao servidor hospedado pela Upstash:

```bash
claude mcp add --transport sse context7 https://mcp.context7.com/sse
```

### Escopo da instalação

Por padrão, o MCP é adicionado apenas ao **projeto atual**. Se quiser que ele esteja disponível em **todos os seus projetos**, use a flag `--scope user`:

```bash
claude mcp add context7 --scope user -- npx -y @upstash/context7-mcp@latest
```

| Escopo | Onde fica disponível |
|--------|---------------------|
| `local` (padrão) | Apenas no diretório/projeto atual |
| `user` | Em todos os projetos do seu usuário |
| `project` | Compartilhado via arquivo no repositório |

### Verificando se o MCP está conectado

Liste os servidores MCP configurados:

```bash
claude mcp list
```

O Context7 deve aparecer com status **`connected`**. Para ver os detalhes de um servidor específico:

```bash
claude mcp get context7
```

Se algo estiver errado, você pode remover e adicionar novamente:

```bash
claude mcp remove context7
```

> 🔄 Caso o Claude Code já estivesse aberto ao adicionar o MCP, encerre a sessão (`/exit`) e reinicie para o servidor ser reconhecido.

---

## Parte 3 — Como usar o Context7 no dia a dia

Para acionar o Context7, basta acrescentar **`use context7`** ao final do seu prompt sempre que precisar de documentação atualizada.

### Exemplos de uso

```
crie um middleware Next.js que valida JWT nos cookies. use context7
```

```
como implementar autenticação JWT no FastAPI? use context7
```

```
mostre como usar React hooks com TypeScript. use context7
```

### Mirando uma biblioteca específica

Você pode indicar exatamente qual biblioteca quer consultar:

```
use library /vercel/next.js para a documentação
```

```
use library /supabase/supabase para a API
```

### Ferramentas que o Context7 expõe

O servidor disponibiliza duas ferramentas principais que o Claude usa automaticamente:

- **`resolve-library-id`** — converte nomes de bibliotecas em IDs internos do Context7.
- **`get-library-docs`** (ou `query-docs`) — busca a documentação propriamente dita.

---

## Solução de problemas

### `npx: command not found`
O Node.js não foi instalado corretamente ou o terminal não foi reiniciado. Feche todos os terminais, abra um novo e tente novamente. Se persistir, reinicie o computador.

### `ERR_MODULE_NOT_FOUND` ao rodar o npx
Tente substituir o `npx` por `bunx` (caso tenha o Bun instalado), ou limpe o cache do npm:

```bash
npm cache clean --force
```

### O Context7 não aparece em `claude mcp list`
- Verifique se o comando `claude mcp add` retornou sucesso (sem erros).
- Reinicie a sessão do Claude Code (`/exit` e abra novamente).
- Se editou o arquivo de configuração manualmente, confira se o JSON está válido (vírgulas, chaves, aspas).

### O arquivo de configuração
Se quiser editar manualmente, o arquivo de config do Claude Code fica em:

- **Windows:** `%USERPROFILE%\.claude.json`
- **macOS/Linux:** `~/.claude.json`

Mas o recomendado é sempre usar `claude mcp add` — ele cuida da formatação correta.

### Biblioteca não encontrada no Context7
Nem todas as bibliotecas estão indexadas. Se o Context7 não encontrar uma lib específica, o Claude vai informar — nesse caso, recorra à documentação oficial direto no site da biblioteca.

---

## Referências

- [Site oficial do Node.js](https://nodejs.org)
- [NVM for Windows (GitHub)](https://github.com/coreybutler/nvm-windows)
- [Context7 (GitHub)](https://github.com/upstash/context7)
- [Documentação oficial do Claude Code](https://docs.claude.com)
- [Microsoft Learn — Node.js no Windows](https://learn.microsoft.com/en-us/windows/dev-environment/javascript/nodejs-on-windows)

---

## Comandos úteis (cola rápida)

```bash
# Verificar versões
node --version
npm --version

# Adicionar Context7 (npx local)
claude mcp add context7 -- npx -y @upstash/context7-mcp@latest

# Adicionar Context7 globalmente (todos os projetos)
claude mcp add context7 --scope user -- npx -y @upstash/context7-mcp@latest

# Listar MCPs configurados
claude mcp list

# Remover um MCP
claude mcp remove context7
```

---

> 📝 **Nota pessoal:** este guia foi criado como referência de consulta. Sinta-se livre para adaptar, copiar e compartilhar.
