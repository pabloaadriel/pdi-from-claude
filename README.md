# 🌊 Sea Hub — Gestão de Tarefas

Sistema interno de gestão de tarefas e acompanhamento de desempenho para o **Sea Hub Coworking**.

---

## 📦 Stack

| Camada      | Tecnologia                       |
|-------------|----------------------------------|
| Frontend    | Next.js 14 (App Router)          |
| Backend     | Firebase (Firestore + Auth + Storage) |
| Estilo      | Tailwind CSS                     |
| Deploy      | Vercel                           |
| Gráficos    | Recharts                         |

---

## 🚀 Configuração Rápida

### 1. Clonar e instalar

```bash
git clone <seu-repositorio>
cd seahub
npm install
```

### 2. Criar projeto no Firebase

1. Acesse [console.firebase.google.com](https://console.firebase.google.com)
2. Clique em **Adicionar projeto** → dê um nome → Criar
3. No menu lateral, ative os serviços:

#### Firebase Authentication
- Menu → **Authentication** → Primeiros passos
- Aba **Sign-in method** → Ativar **E-mail/senha**

#### Cloud Firestore
- Menu → **Firestore Database** → Criar banco de dados
- Escolha **Modo de produção**
- Selecione a região (ex: `southamerica-east1`)

#### Firebase Storage
- Menu → **Storage** → Primeiros passos
- Modo de produção → Selecione a mesma região

### 3. Obter credenciais do Firebase

1. No Console Firebase → ⚙️ **Configurações do projeto** → **Seus apps**
2. Clique no ícone `</>` (Web) → Registrar app → copie o `firebaseConfig`

### 4. Configurar variáveis de ambiente

```bash
cp .env.local.example .env.local
```

Edite `.env.local` com seus valores:

```env
NEXT_PUBLIC_FIREBASE_API_KEY=AIzaSy...
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=seu-projeto.firebaseapp.com
NEXT_PUBLIC_FIREBASE_PROJECT_ID=seu-projeto
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=seu-projeto.appspot.com
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=123456789
NEXT_PUBLIC_FIREBASE_APP_ID=1:123456789:web:abc123
```

### 5. Publicar regras do Firestore e Storage

Instale o Firebase CLI:

```bash
npm install -g firebase-tools
firebase login
firebase use --add   # selecione seu projeto
```

Publique as regras:

```bash
firebase deploy --only firestore:rules
firebase deploy --only firestore:indexes
firebase deploy --only storage
```

### 6. Popular usuários iniciais (seed)

```bash
npm install firebase-admin --save-dev
```

1. No Console Firebase → **Configurações do projeto** → **Contas de serviço**
2. Clique em **Gerar nova chave privada** → salve como `scripts/serviceAccountKey.json`
3. Execute:

```bash
node scripts/seed.js
```

Isso cria automaticamente:
- Todos os usuários no Firebase Auth (com senha interna)
- Documentos na coleção `users` do Firestore
- Tarefas de exemplo para cada colaborador

> ⚠️ **Importante:** O arquivo `serviceAccountKey.json` está no `.gitignore`. Nunca o commite.

### 7. Rodar localmente

```bash
npm run dev
```

Acesse: [http://localhost:3000](http://localhost:3000)

---

## 🌐 Deploy na Vercel

### Via CLI

```bash
npm install -g vercel
vercel
```

### Via GitHub (recomendado)

1. Suba o projeto para um repositório no GitHub
2. Acesse [vercel.com](https://vercel.com) → **New Project** → Importe o repositório
3. Em **Environment Variables**, adicione todas as variáveis do `.env.local`
4. Clique em **Deploy**

---

## 🔐 Sistema de Login

O login é feito **apenas com email** — sem senha visível ao usuário.

**Fluxo:**
1. Usuário digita o email
2. Sistema verifica se o email está na lista autorizada
3. Se autorizado → autentica automaticamente com o Firebase Auth
4. Se não autorizado → exibe mensagem de erro

**Emails autorizados:**

| Email | Papel |
|-------|-------|
| fulvio@seahubcoworking.com.br | Colaborador |
| regis@seahubcoworoking.com.br | Colaborador |
| diegosena@seahubcoworking.com.br | Colaborador |
| natha@seahubcoworking.com.br | Colaborador |
| socorro@seahubcoworking.com.br | Colaborador |
| meduarda@seahubcoworking.com.br | Colaborador |
| guilherme@seahubcoworking.com.br | Gestor |
| laramartins@segantiniconsultoria.com | Gestor |
| pabloaadriel@gmail.com | Gestor |

---

## 📁 Estrutura do Projeto

```
seahub/
├── src/
│   ├── app/
│   │   ├── page.tsx                    # Login
│   │   ├── layout.tsx                  # Root layout
│   │   ├── globals.css
│   │   └── (app)/                      # Rotas protegidas
│   │       ├── layout.tsx              # Auth guard + sidebar
│   │       ├── dashboard/page.tsx      # Dashboard pessoal
│   │       ├── tarefas/page.tsx        # Minhas Tarefas
│   │       └── gestao/page.tsx         # Painel do Gestor
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Sidebar.tsx
│   │   │   ├── AppShell.tsx
│   │   │   └── AuthProvider.tsx
│   │   ├── ui/
│   │   │   ├── Modal.tsx
│   │   │   ├── Badge.tsx
│   │   │   ├── ProgressBar.tsx
│   │   │   └── EmptyState.tsx
│   │   ├── tasks/
│   │   │   ├── TaskCard.tsx
│   │   │   ├── TaskDetailModal.tsx
│   │   │   ├── TaskFormModal.tsx
│   │   │   └── PilarSection.tsx
│   │   └── dashboard/
│   │       ├── StatCard.tsx
│   │       ├── AdesaoCard.tsx
│   │       └── Charts.tsx
│   ├── hooks/
│   │   └── useAuth.tsx
│   ├── lib/
│   │   ├── firebase.ts
│   │   ├── auth.ts
│   │   ├── tasks.ts
│   │   └── users.ts
│   └── types/
│       └── index.ts
├── scripts/
│   └── seed.js
├── firestore.rules
├── firestore.indexes.json
├── storage.rules
├── firebase.json
├── .env.local.example
├── package.json
├── tailwind.config.ts
└── next.config.js
```

---

## 📊 Regras de Negócio

### Status Visual das Tarefas

| Condição | Status Visual |
|----------|---------------|
| Não concluída + prazo não passou | A FAZER / EM ANDAMENTO |
| Não concluída + prazo passou | EM ATRASO |
| Concluída + dentro do prazo | SEGUINDO O PLANO |
| Concluída + fora do prazo | FECHADA FORA DO PLANO |

### Critério de Adesão

Um colaborador está **em adesão** quando:
- Progresso geral ≥ **80%**
- Progresso em cada pilar ≥ **70%**

### Comprovação

- **Obrigatória** para marcar tarefa como concluída
- Aceita: link (URL) ou upload de arquivo (imagem/PDF, máx 10MB)
- Armazenada no Firebase Storage

---

## 🛠️ Comandos Úteis

```bash
npm run dev          # Desenvolvimento local
npm run build        # Build de produção
npm run start        # Servidor de produção local
firebase deploy      # Deploy das regras Firebase
node scripts/seed.js # Popular banco com dados iniciais
```

---

## ❓ Problemas Comuns

**"Permission denied" no Firestore**
→ Verifique se as regras foram publicadas: `firebase deploy --only firestore:rules`

**Erro de CORS no Storage**
→ Configure o CORS no Firebase Storage via `gsutil cors set cors.json gs://seu-bucket`

**Login não funciona**
→ Verifique se o método Email/Senha está ativado no Firebase Auth

**Variáveis de ambiente não carregam na Vercel**
→ Adicione todas as variáveis `NEXT_PUBLIC_*` nas configurações do projeto na Vercel e faça um novo deploy

---

## 📝 Licença

Projeto privado — Sea Hub Coworking © 2024
