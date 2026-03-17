# 🚀 Guia de Deploy — EscalaFé PWA
## Firebase Hosting + Authentication + Firestore

---

## ESTRUTURA DE ARQUIVOS

```
escalafe/
├── index.html       ← App principal (tela de login + sistema)
├── manifest.json    ← Configuração do PWA
├── sw.js            ← Service Worker (modo offline)
├── icon-192.png     ← Ícone do app (192×192px) ← VOCÊ CRIA
├── icon-512.png     ← Ícone do app (512×512px)  ← VOCÊ CRIA
└── .firebaserc      ← Criado pelo Firebase CLI
```

---

## PASSO 1 — Criar projeto no Firebase (gratuito)

1. Acesse https://console.firebase.google.com
2. Clique em **"Criar um projeto"**
3. Dê um nome: `escalafe-suaigreja`
4. Desative o Google Analytics (opcional)
5. Clique em **"Criar projeto"**

---

## PASSO 2 — Ativar Authentication

1. No console do Firebase, clique em **"Authentication"** (menu lateral)
2. Clique em **"Começar"**
3. Aba **"Sign-in method"** → habilite **"E-mail/senha"** → Salvar

---

## PASSO 3 — Ativar Firestore Database

1. Clique em **"Firestore Database"** → **"Criar banco de dados"**
2. Selecione **"Iniciar no modo de produção"**
3. Escolha a região: `southamerica-east1` (São Paulo)
4. Clique em **"Ativar"**

### Regras de segurança do Firestore
Vá em **Firestore → Regras** e cole:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Usuários só leem/editam o próprio perfil
    match /usuarios/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth.uid == userId;
    }

    // Admins gerenciam tudo; líderes editam escala; obreiros leem
    match /membros/{doc} {
      allow read: if request.auth != null;
      allow write: if get(/databases/$(database)/documents/usuarios/$(request.auth.uid)).data.role in ['admin'];
    }

    match /ministerios/{doc} {
      allow read: if request.auth != null;
      allow write: if get(/databases/$(database)/documents/usuarios/$(request.auth.uid)).data.role in ['admin'];
    }

    match /cultos/{doc} {
      allow read: if request.auth != null;
      allow write: if get(/databases/$(database)/documents/usuarios/$(request.auth.uid)).data.role in ['admin'];
    }

    match /escala/{doc} {
      allow read: if request.auth != null;
      allow write: if get(/databases/$(database)/documents/usuarios/$(request.auth.uid)).data.role in ['admin', 'lider'];
    }

    match /disponibilidade/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth.uid == userId
        || get(/databases/$(database)/documents/usuarios/$(request.auth.uid)).data.role == 'admin';
    }
  }
}
```

---

## PASSO 4 — Copiar as credenciais do Firebase

1. Na página inicial do projeto, clique no ícone **"</>"** (Web)
2. Registre o app com o nome `escalafe`
3. Copie o objeto `firebaseConfig` que aparece
4. **No arquivo `index.html`**, substitua o bloco:

```javascript
const firebaseConfig = {
  apiKey:            "COLE_SEU_API_KEY_AQUI",   // ← substitua
  authDomain:        "SEU_PROJETO.firebaseapp.com",
  projectId:         "SEU_PROJETO",
  storageBucket:     "SEU_PROJETO.appspot.com",
  messagingSenderId: "SEU_SENDER_ID",
  appId:             "SEU_APP_ID"
};
```

Pelos valores reais copiados do console.

---

## PASSO 5 — Criar os ícones do app

Crie dois arquivos PNG com a cruz/logo da sua igreja:
- `icon-192.png` — 192×192 pixels
- `icon-512.png` — 512×512 pixels

Dica: use https://favicon.io ou https://www.canva.com para criar gratuitamente.

---

## PASSO 6 — Instalar Firebase CLI e fazer deploy

Abra o terminal (cmd / PowerShell / Terminal):

```bash
# Instalar Node.js se não tiver: https://nodejs.org

# 1. Instalar Firebase CLI
npm install -g firebase-tools

# 2. Fazer login no Firebase
firebase login

# 3. Entrar na pasta do projeto
cd caminho/para/escalafe

# 4. Inicializar Firebase Hosting
firebase init hosting
# → "Use an existing project" → selecione seu projeto
# → "What do you want to use as your public directory?" → . (ponto)
# → "Configure as a single-page app?" → N
# → "Set up automatic builds with GitHub?" → N

# 5. Deploy!
firebase deploy --only hosting
```

Após o deploy, seu app estará em:
👉 `https://escalaigreja-1aa79.web.app`

---

## PASSO 7 — Criar usuários (líderes e obreiros)

### Via Firebase Console (recomendado para o primeiro admin):

1. Vá em **Authentication → Users → Add user**
2. Insira e-mail e senha de cada líder
3. Copie o **UID** do usuário criado

### Criar perfil no Firestore:

1. Vá em **Firestore → Dados → + Iniciar coleção**
2. Coleção: `usuarios`
3. ID do documento: cole o **UID** do usuário
4. Campos:
   ```
   nome    (string)  → "Carlos Lima"
   role    (string)  → "admin" | "lider" | "obreiro"
   email   (string)  → "carlos@email.com"
   memberId (string) → ID do membro no sistema (ex: "m2")
   ```

### Roles disponíveis:
| Role | Permissões |
|------|-----------|
| `admin` | Acesso total: cadastra, edita tudo |
| `lider` | Edita escala apenas do seu ministério |
| `obreiro` | Visualiza tudo, edita própria disponibilidade |

---

## PASSO 8 — Instalar o app no celular (PWA)

### Android (Chrome):
1. Acesse `https://SEU-PROJETO.web.app`
2. Chrome exibirá um banner **"Instalar EscalaFé"**
3. Ou toque nos 3 pontos → "Adicionar à tela inicial"

### iPhone (Safari):
1. Acesse `https://SEU-PROJETO.web.app` no Safari
2. Toque no botão **Compartilhar** (quadrado com seta)
3. Role para baixo → **"Adicionar à Tela de Início"**
4. Confirme → o ícone aparecerá como um app!

---

## LIMITES DO PLANO GRATUITO (Firebase Spark)

| Recurso | Limite gratuito |
|---------|----------------|
| Authentication | Ilimitado |
| Firestore leituras | 50.000/dia |
| Firestore escritas | 20.000/dia |
| Hosting tráfego | 10 GB/mês |
| Hosting armazenamento | 1 GB |

✅ **Suficiente para uma igreja de qualquer tamanho!**

---

## DÚVIDAS FREQUENTES

**Q: O app funciona sem internet?**
A: Sim! O Service Worker faz cache do app. Dados em tempo real requerem conexão.

**Q: Posso ter vários admins?**
A: Sim, basta criar usuários com `role: "admin"` no Firestore.

**Q: Como redefinir a senha de um líder?**
A: Na tela de login, clique em "Esqueci minha senha" — o Firebase envia o e-mail automaticamente.

**Q: Como atualizar o app depois?**
A: Edite os arquivos e rode `firebase deploy --only hosting` novamente.

---

## SUPORTE

Gerado com EscalaFé — Sistema de Gestão de Obreiros
