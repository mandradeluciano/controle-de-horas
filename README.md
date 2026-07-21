# Ponto da Babá

App para apontamento de horas de babá, com cálculo de horas normais, horas extras
e valor a pagar. Relatórios diário, semanal, quinzenal e mensal. Modo de ponto ao
vivo (chegada / encerrar) e registro manual.

Roda **online** com Firebase (login por e-mail e dados sincronizados entre
aparelhos). Sem configurar o Firebase, ele funciona em **modo local** neste
navegador — útil para testar antes de publicar.

---

## Arquivos

- `index.html` — o app inteiro (interface + lógica + integração Firebase).
- `firestore.rules` — regras de segurança do banco (cada pessoa só vê os próprios dados).
- `firebase.json` — configuração do Firebase Hosting.
- `README.md` — este guia.

---

## Parte 1 — Criar o projeto no Firebase (uma vez)

1. Acesse https://console.firebase.google.com e clique em **Adicionar projeto**.
2. Dê um nome (ex.: `ponto-da-baba`) e conclua a criação.
3. No menu **Criação → Authentication → Começar**, aba **Sign-in method**,
   ative **E-mail/senha**.
4. No menu **Criação → Firestore Database → Criar banco de dados**,
   escolha **Modo de produção** e uma região (ex.: `southamerica-east1`).
5. No topo (ícone de engrenagem) **Configurações do projeto → Seus apps**,
   clique em **</>** (Web), registre o app e **copie o objeto `firebaseConfig`**.

## Parte 2 — Colar a config no app

Abra `index.html`, procure por `firebaseConfig` (perto do início) e substitua
os valores `COLE_...` / `SEU_PROJETO` pelos dados que você copiou:

```js
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "ponto-da-baba.firebaseapp.com",
  projectId: "ponto-da-baba",
  storageBucket: "ponto-da-baba.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abcdef..."
};
```

## Parte 3 — Subir para o GitHub

```bash
# dentro da pasta com os arquivos
git init
git add .
git commit -m "Ponto da Babá - versão inicial"
git branch -M main
git remote add origin https://github.com/SEU_USUARIO/ponto-da-baba.git
git push -u origin main
```

> Dica: como a `apiKey` do Firebase Web fica visível no código do navegador,
> tudo bem versioná-la. Quem protege os dados são as **regras do Firestore**
> (`firestore.rules`), não o segredo da chave.

## Parte 4 — Publicar com Firebase Hosting

Precisa do Node.js instalado (https://nodejs.org).

```bash
npm install -g firebase-tools     # instala a CLI (uma vez)
firebase login                    # abre o navegador para entrar
firebase use --add                # escolha o projeto criado na Parte 1

# publica as regras do banco e o site
firebase deploy --only firestore:rules
firebase deploy --only hosting
```

Ao final, a CLI mostra a URL pública, tipo:
`https://ponto-da-baba.web.app` — abra no celular e no computador com o mesmo
login para ver os dados sincronizados.

### (Opcional) Deploy automático a cada push no GitHub

```bash
firebase init hosting:github
```

Isso cria um workflow do GitHub Actions que publica sozinho sempre que você
fizer `git push` na branch `main`.

---

## Alternativa: hospedar no GitHub Pages

Se preferir hospedar o site no GitHub Pages e usar o Firebase só como banco:

1. No repositório: **Settings → Pages → Branch: `main` / root → Save**.
2. Aguarde a URL `https://SEU_USUARIO.github.io/ponto-da-baba/`.
3. No Firebase: **Authentication → Settings → Authorized domains → Add domain**
   e adicione `SEU_USUARIO.github.io` (senão o login é bloqueado).

O Firestore funciona igual; muda apenas onde o `index.html` fica hospedado.

---

## Como os dados são guardados

Cada usuário tem um documento em `usuarios/{uid}` no Firestore, com os campos
`config`, `registros` e `sessao` (guardados como texto JSON). As regras garantem
que ninguém acesse os dados de outra pessoa.
