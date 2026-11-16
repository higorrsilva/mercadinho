# 🛒 Mercadinho Seu João

Sistema web de **controle de estoque** para o Mercadinho Seu João: cadastro de produtos, entradas e saídas, controle de validade por lote, alertas e relatórios, com **níveis de usuário e permissões por tela**.

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-backend-000000?logo=flask&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL-3ECF8E?logo=supabase&logoColor=white)

---

## ✨ Funcionalidades

- **Painel** com visão geral do estoque e alertas
- **Produtos** e categorias
- **Entradas** e **Saídas** de estoque, com controle por **lote e validade**
- Baixa automática pela regra **FEFO** (sai primeiro o lote com validade mais próxima)
- **Alertas** de estoque baixo e validade próxima
- **Relatórios**
- **Fornecedores**
- **Caixa**
- **Usuários e permissões** (somente o Dono)

## 🔐 Níveis de usuário e permissões

| Perfil | Acesso |
|---|---|
| **DONO** (super admin) | Acessa todas as telas, cria e edita usuários, faz ajustes de estoque e exclui movimentações |
| **FUNCIONARIO** | Acessa apenas as telas liberadas pelo Dono para ele |

As telas que podem ser liberadas para um funcionário são: `painel`, `caixa`, `produtos`, `entradas`, `saidas`, `relatorios` e `fornecedores`.

Como funciona:

- O login gera um **token assinado**, válido por 8 horas, que o front envia em todas as chamadas à API.
- As permissões são verificadas **no servidor** (um funcionário recebe erro 403 se chamar uma rota que não pode, mesmo digitando o endereço direto).
- O menu lateral mostra só as telas permitidas, e cada página redireciona quem não tem acesso.
- As senhas são salvas com **hash**.
- Usuários são **inativados**, nunca excluídos.

## 🧰 Tecnologias

- **Front-end:** HTML, CSS e JavaScript
- **Back-end:** Python + Flask
- **Banco de dados:** Supabase (PostgreSQL)

## 📁 Estrutura do projeto

```
mercadinho/
├── app.py               # Servidor Flask (rotas, autenticação, regras de estoque)
├── conexao.py           # Conexão com o Supabase
├── requirements.txt     # Dependências Python
├── *.html               # Páginas (painel, produtos, entradas, saídas, etc.)
├── main.js              # Script principal do front-end
├── css/                 # Estilos
├── js/                  # Lógica de cada página e controle de sessão
├── services/api/        # Cliente de API (envio do token, tratamento de erros)
├── assets/              # Imagens e ícones
├── tests/               # Testes
└── CLAUDE.md            # Contexto e convenções do projeto
```

## 🚀 Como rodar o projeto

### Pré-requisitos

- [Python 3.10 ou superior](https://www.python.org/downloads/)
- [Git](https://git-scm.com/)
- Uma conta e um projeto no [Supabase](https://supabase.com/)

### 1. Clonar o repositório

```bash
git clone https://github.com/SEU_USUARIO/mercadinho.git
cd mercadinho
```

### 2. Criar e ativar o ambiente virtual (`.venv`)

**Windows (PowerShell):**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

> Se o PowerShell bloquear a ativação, rode antes (vale só para essa janela):
> `Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned`

**Linux / macOS:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Quando o ambiente estiver ativo, o terminal mostra `(.venv)` no começo da linha.

### 3. Instalar as dependências

```bash
pip install -r requirements.txt
```

### 4. Configurar o arquivo `.env`

Crie um arquivo chamado `.env` na raiz do projeto (você pode copiar o modelo `.env.example`):

```env
SUPABASE_URL=https://SEU-PROJETO.supabase.co
SUPABASE_KEY=sua-chave-do-supabase
SECRET_KEY=um-texto-longo-e-aleatorio
```

- **`SUPABASE_URL` e `SUPABASE_KEY`:** no painel do Supabase, em **Project Settings → API**.
- **`SECRET_KEY`:** assina os tokens de login. Para gerar uma:

  ```bash
  python -c "import secrets; print(secrets.token_hex(32))"
  ```

> ⚠️ **Nunca envie o `.env` para o GitHub.** Ele já está no `.gitignore`.

### 5. Preparar o banco de dados

No Supabase, abra o **SQL Editor** e execute o comando que cria a coluna de permissões:

```sql
ALTER TABLE public.usuario
ADD COLUMN IF NOT EXISTS permissoes text[] NOT NULL DEFAULT ARRAY[]::text[];
```

Se já houver funcionários cadastrados, libere para eles as telas padrão:

```sql
UPDATE public.usuario
SET permissoes = ARRAY['painel','caixa','entradas','saidas','produtos']
WHERE tipo = 'FUNCIONARIO';
```

### 6. Criar o primeiro usuário Dono

O cadastro público foi removido, então o primeiro Dono é criado direto no banco. No **Table Editor** do Supabase, na tabela `usuario`, insira uma linha com:

| Campo | Valor |
|---|---|
| `nome` | seu nome |
| `login` | seu login (ex.: e-mail) |
| `senha` | uma senha qualquer (ela é convertida em hash no primeiro login) |
| `tipo` | `DONO` |
| `ativo` | `true` |

Depois, os demais usuários são criados pelo próprio sistema, na tela **Usuários**.

### 7. Iniciar o servidor

```bash
python app.py
```

Abra no navegador: **http://127.0.0.1:8000**

Para parar o servidor, use `Ctrl + C` no terminal. Sempre que alterar o código do `app.py`, pare e inicie de novo.

## 🧪 Testes

```bash
python -m unittest
```

> ⚠️ Parte dos testes grava dados no Supabase. Prefira usar um projeto Supabase separado para testes, e não o banco de uso real.

## 🛠️ Problemas comuns

| Problema | Solução |
|---|---|
| Telas sem dados ou erro de acesso | Entre de novo no sistema e recarregue com `Ctrl + F5` |
| Erro ao subir o servidor | Confira se o `.venv` está ativo e se o `.env` tem as três variáveis |
| `Credenciais inválidas` | Confira o login (sem diferença de maiúsculas) e se o usuário está `ativo` |
| `Sem permissão` ao abrir uma tela | O Dono precisa liberar a tela para o funcionário, na página **Usuários** |
| O PowerShell não ativa o `.venv` | Rode `Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned` e tente de novo |

Projeto acadêmico de Sistemas de Informação.
