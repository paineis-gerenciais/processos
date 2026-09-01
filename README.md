# ProcessFlow — publicar no GitHub Pages + compartilhar dados pelo Firestore

Este pacote tem 4 arquivos:

| Arquivo | Para que serve |
|---|---|
| `index.html` | O aplicativo inteiro (dashboard + editor BPMN). É o único arquivo que precisa ser aberto. |
| `logo.png` | A logo usada no canto superior esquerdo e no cabeçalho do PDF. |
| `firestore.rules` | Regras de acesso do banco de dados compartilhado. |
| `firebase.json` | Arquivo de configuração usado apenas se você for publicar as regras pelo terminal (opcional). |

**Sem configurar nada**, o `index.html` já funciona sozinho: basta abrir o arquivo (ou publicá-lo no GitHub Pages) e cada pessoa terá seus próprios processos salvos no navegador dela (não compartilhado). As seções abaixo mostram como ligar o Firestore para que **todo o grupo veja e edite os mesmos processos, em tempo real**.

---

## Parte 1 — Criar o banco de dados compartilhado (Firestore)

1. Acesse [console.firebase.google.com](https://console.firebase.google.com) e faça login com uma conta Google.
2. Clique em **"Adicionar projeto"**, dê um nome (ex.: `processflow-grupo`) e conclua a criação (pode desativar o Google Analytics, não é necessário).
3. No menu lateral, vá em **Build > Firestore Database** e clique em **"Criar banco de dados"**.
   - Escolha a localização mais próxima do grupo (ex.: `southamerica-east1` para o Brasil).
   - Pode iniciar em **modo de produção** — as regras corretas estão no arquivo `firestore.rules` deste pacote e você vai colá-las no passo 5.
4. Ainda no Firebase Console, clique no ícone de **engrenagem (⚙) > Configurações do projeto**.
5. Role até **"Seus apps"** e clique no ícone **`</>`** (Web) para registrar um app web.
   - Dê um apelido (ex.: `processflow-web`) — não precisa marcar Firebase Hosting.
   - Clique em **Registrar app**. O Firebase vai mostrar um bloco de código parecido com este:

   ```js
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "processflow-grupo.firebaseapp.com",
     projectId: "processflow-grupo",
     storageBucket: "processflow-grupo.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef123456"
   };
   ```

   **Copie esses 6 valores** — você vai colá-los no `index.html` no próximo passo.

---

## Parte 2 — Colar a configuração no `index.html`

1. Abra o `index.html` em um editor de texto (Bloco de Notas, VS Code, etc.).
2. Procure por este bloco perto do topo do arquivo (dentro de `<head>`):

   ```js
   window.PROCESSFLOW_FIREBASE_CONFIG = {
     apiKey: "COLE_AQUI",
     authDomain: "SEU-PROJETO.firebaseapp.com",
     projectId: "SEU-PROJETO",
     storageBucket: "SEU-PROJETO.appspot.com",
     messagingSenderId: "COLE_AQUI",
     appId: "COLE_AQUI"
   };
   ```

3. Substitua os 6 valores pelos que você copiou do Firebase Console no passo anterior. Salve o arquivo.

> Se esse bloco continuar com `"COLE_AQUI"`, o app detecta isso automaticamente e usa apenas o navegador local (modo offline/individual) — ele nunca trava por falta de configuração.

---

## Parte 3 — Publicar as regras de acesso do Firestore

Isso libera o app para ler e escrever na coleção `processes` (sem isso, o Firestore bloqueia tudo por padrão).

### Opção A — pelo Console (mais simples, sem instalar nada)

1. No Firebase Console, vá em **Build > Firestore Database > Regras**.
2. Apague o conteúdo e cole o conteúdo do arquivo `firestore.rules` deste pacote.
3. Clique em **Publicar**.

### Opção B — pelo terminal, com o Firebase CLI (opcional)

```bash
npm install -g firebase-tools
firebase login
# na pasta onde estão firebase.json e firestore.rules:
firebase use --add        # escolha o projeto que você criou
firebase deploy --only firestore:rules
```

---

## Parte 4 — Publicar o site no GitHub Pages

1. Crie uma conta em [github.com](https://github.com) se ainda não tiver.
2. Clique em **New repository** (Novo repositório).
   - Nome sugerido: `processflow`
   - Marque como **Public** (o GitHub Pages gratuito exige repositório público).
   - Não marque "Add a README" (já temos um).
3. Envie os arquivos deste pacote (`index.html`, `logo.png`, `firestore.rules`, `firebase.json`, `README.md`) para o repositório:

   **Pelo site do GitHub (mais simples):**
   - Na página do repositório, clique em **"Add file" > "Upload files"**.
   - Arraste os arquivos, escreva uma mensagem de commit (ex.: "Primeira versão") e clique em **"Commit changes"**.

   **Ou pelo terminal:**
   ```bash
   git init
   git add index.html logo.png firestore.rules firebase.json README.md
   git commit -m "Primeira versão do ProcessFlow"
   git branch -M main
   git remote add origin https://github.com/SEU-USUARIO/processflow.git
   git push -u origin main
   ```

4. No repositório, vá em **Settings > Pages**.
5. Em **"Build and deployment" > Source**, escolha **"Deploy from a branch"**.
6. Em **Branch**, selecione `main` e a pasta `/ (root)`. Clique em **Save**.
7. Aguarde 1–2 minutos. O GitHub mostrará o link publicado, algo como:

   ```
   https://SEU-USUARIO.github.io/processflow/
   ```

8. Abra esse link — o `index.html` carrega automaticamente. Compartilhe esse link com o grupo.

---

## Parte 5 — Testando

- Abra o link em dois navegadores (ou computadores) diferentes.
- Crie um processo em um deles e clique em **Salvar**.
- No outro navegador, volte ao painel (dashboard) — o novo processo deve aparecer sozinho, sem precisar recarregar a página (o app escuta as mudanças do Firestore em tempo real).
- Se algo não aparecer, abra o Console do navegador (F12) e veja se há algum erro relacionado a "Firebase" ou "Firestore" — geralmente é a configuração colada errada ou as regras não publicadas.

---

## Notas importantes

- **Sem autenticação de usuários.** Qualquer pessoa com o link do site (e, portanto, com a configuração do Firebase visível no código-fonte) pode ler e escrever os processos. Isso é intencional para manter o app simples, mas significa que não é indicado para dados sensíveis — use apenas dentro de um grupo de confiança.
- **Trocar a logo:** basta substituir o arquivo `logo.png` por outro com o mesmo nome (qualquer proporção funciona; a logo se ajusta automaticamente na barra lateral, na barra de ferramentas e no cabeçalho do PDF).
- **Sem Firestore configurado:** o app continua funcionando 100% localmente (por navegador), incluindo os dois processos de exemplo pré-carregados na primeira vez que é aberto.
- **Custos:** o Firestore tem uma camada gratuita generosa (milhares de leituras/escrituras por dia) — para um grupo pequeno de trabalho, dificilmente vai gerar custo.
