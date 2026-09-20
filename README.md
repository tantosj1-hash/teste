# Site — Igreja Luz Para Os Povos (Jardim da Luz)

Site institucional com galeria de fotos/vídeos, agenda de eventos com calendário
interativo, área de bazar (sem preços) e Área de Líderes com acesso restrito
por e-mail para editar cultos e agenda.

**Stack:** React 18 + Vite + React Router + Framer Motion + Firebase
(Auth com login Google, Firestore e Storage), hospedado no Firebase Hosting.

## Cores do site
Laranja (`#FF7A1A`), preto (`#121212`) e branco — definidas em `src/styles/global.css`.

---

## 1. Rodando localmente

```bash
npm install
npm run dev
```

O site abre em `http://localhost:5173`. Sem configurar o Firebase (passo 2),
os dados de cultos usam o valor padrão de `src/lib/igreja.config.js`, e agenda,
galeria e bazar aparecem vazios — mas o site já funciona visualmente.

---

## 2. Configurando o Firebase (obrigatório para a Área de Líderes funcionar)

1. Acesse [console.firebase.google.com](https://console.firebase.google.com) e crie um projeto
   (sugestão de ID: `jardim-da-luz`).
2. Em **Build > Authentication > Sign-in method**, ative o provedor **Google**.
3. Em **Build > Firestore Database**, clique em "Criar banco de dados" (modo produção).
4. Em **Build > Storage**, clique em "Começar" para ativar o Storage.
5. Em **Configurações do projeto > Seus apps**, adicione um app Web e copie as
   chaves geradas.
6. Copie `.env.example` para `.env` e cole as chaves:

```bash
cp .env.example .env
```

7. Edite `src/lib/igreja.config.js` e confirme a lista `EMAILS_AUTORIZADOS` com
   os e-mails do Google que poderão acessar a Área de Líderes (`/lideres`).
   Por padrão já está com `natanaraujo.gg@gmail.com`.
8. **Importante:** replique a mesma lista de e-mails em `firestore.rules` e
   `storage.rules` (dentro da função `ehLider()`), pois essa é a proteção real
   contra edição pública — o que está no navegador pode ser inspecionado, mas
   as Regras do Firebase são a barreira de segurança de verdade.

---

## 3. Publicando as regras de segurança e o site (Firebase Hosting)

```bash
npm install -g firebase-tools   # se ainda não tiver
firebase login
firebase deploy --only firestore:rules,storage:rules
npm run build
firebase deploy --only hosting
```

Ou, de uma vez: `npm run deploy` (já faz `build` + `deploy --only hosting`).

---

## 4. Conectando o domínio luzjardimdaluz.com.br

1. No Console Firebase, vá em **Hosting > Adicionar domínio personalizado**.
2. Digite `luzjardimdaluz.com.br` (e opcionalmente `www.luzjardimdaluz.com.br`).
3. O Firebase vai mostrar registros DNS (tipo `A` e/ou `TXT`) para você
   cadastrar no painel onde o domínio foi comprado (Registro.br, GoDaddy, etc.).
4. Após propagar o DNS (pode levar algumas horas), o Firebase emite o
   certificado SSL automaticamente e o site passa a responder em
   `https://luzjardimdaluz.com.br`.

---

## 5. Como usar a Área de Líderes

- Acesse `luzjardimdaluz.com.br/lideres`.
- Clique em "Entrar com Google" e use um e-mail da lista de autorizados.
- No painel, use as abas:
  - **Cultos** — cadastra/edita/exclui os horários de culto exibidos na home.
  - **Agenda** — cadastra/edita/exclui os eventos (aparecem no calendário e nas listas de eventos).
  - **Galeria** — envia fotos e vídeos (exibidos publicamente em `/galeria` e na home).
  - **Bazar** — envia fotos das peças do bazar (sem campo de valor/preço).
  - **Líderes** — mostra quem tem acesso e como adicionar/remover um líder.
- Qualquer visitante sem login, ou logado com e-mail não autorizado, só pode
  **visualizar** o site — nunca editar, incluir ou remover conteúdo.

## 6. Adicionar ou remover um líder autorizado

1. Edite a lista `EMAILS_AUTORIZADOS` em `src/lib/igreja.config.js`.
2. Edite a mesma lista dentro da função `ehLider()` em `firestore.rules` e em `storage.rules`.
3. Rode novamente:
   ```bash
   firebase deploy --only firestore:rules,storage:rules
   npm run deploy
   ```

## 7. Localização no rodapé da página inicial

O link do Google Maps da igreja está em `src/lib/igreja.config.js`
(`googleMapsUrl`) e já aponta para
`https://maps.app.goo.gl/CywHiDgrr6vsxLTs8`. O mapa exibido na home é um mapa
incorporado do Google (iframe) baseado nas coordenadas `mapa.lat` / `mapa.lng`
do mesmo arquivo — não precisa de chave de API. O selo "Ver no Google Maps"
sobre o mapa abre o link oficial.

---

## Estrutura de pastas

```
src/
  components/       Header, Footer, Calendário, RotaProtegida
  context/          Autenticação (login Google + verificação de e-mail autorizado)
  lib/              Configuração do Firebase, dados da igreja, funções de dados (Firestore/Storage)
  pages/            Início, Sobre, Agenda, Galeria, Bazar
  pages/admin/      Login e Painel da Área de Líderes (+ abas de edição)
  styles/           CSS global (cores laranja/preto/branco, layout responsivo)
firestore.rules     Regras de segurança do banco de dados
storage.rules       Regras de segurança dos arquivos (fotos/vídeos)
```
