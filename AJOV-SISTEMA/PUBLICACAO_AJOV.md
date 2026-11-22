# Publicação AJOV-SISTEMA — Netlify + Render + PostgreSQL

## Arquitectura de publicação
- **Netlify:** frontend estático, diretório de publicação `frontend/`.
- **Render:** backend Node/Express, diretório raiz `backend/`.
- **Render PostgreSQL:** armazenamento dos cadastros.
- **Nodemailer/Gmail:** notificações de email configuradas no backend.

## 1. Preparação e segurança
- Não publique ficheiros `.env`, credenciais, tokens, bases de dados locais, uploads ou backups.
- Se alguma senha de aplicação já foi exposta anteriormente, revogue-a e gere uma nova.
- O ficheiro `backend/.env.example` é apenas um modelo; crie localmente `backend/.env`.

## 2. Criar a base PostgreSQL
No Render, crie uma base PostgreSQL e copie a URL de ligação privada para as variáveis de ambiente do backend. Nunca partilhe essa URL.

## 3. Publicar o backend no Render
Crie um Web Service ligado ao repositório:
- Root Directory: `backend`
- Runtime: Node
- Build Command: `npm install --omit=dev`
- Start Command: `npm start`

Configure estas variáveis no painel Render:
- `NODE_ENV=production`
- `DATABASE_URL` — URL do PostgreSQL
- `DATABASE_SSL=true` (ajuste se o serviço de banco usar ligação sem TLS)
- `CORS_ORIGIN` — URL exata do site Netlify, sem wildcard
- `EMAIL_USER` — email remetente
- `EMAIL_PASSWORD` — palavra-passe de aplicação Gmail
- `EMAIL_DESTINATARIO` — email que recebe notificações
- `ADMIN_API_TOKEN` — token longo e aleatório para endpoints administrativos

Não defina `PORT` manualmente; o Render fornece essa variável.

Após publicar, teste `https://SEU-BACKEND.onrender.com/api/health`. A resposta esperada indica `status: ok` e `database: connected`.

## 4. Configurar o frontend para usar o backend publicado
Abra `frontend/js/config.js` e substitua o URL placeholder pelo endereço real do backend, por exemplo:
```js
window.__API_BASE_URL__ = 'https://SEU-BACKEND.onrender.com';
```
Este ficheiro é público. Não coloque nele `DATABASE_URL`, `EMAIL_PASSWORD` ou `ADMIN_API_TOKEN`.

## 5. Publicar no Netlify
Importe o repositório no Netlify. O `netlify.toml` já configura:
- Publish directory: `frontend`
- Build command: vazio (site estático)

Depois de publicar, actualize `CORS_ORIGIN` no Render com o URL Netlify definitivo e faça redeploy do backend se necessário.

## 6. Testes pós-publicação
1. Abra o site Netlify e verifique se o layout, logo e campos carregam.
2. Envie um cadastro com dados de teste.
3. Confirme que a API responde e o registo foi guardado no PostgreSQL.
4. Confirme o email de notificação (incluindo Spam). Se email não estiver configurado, o sistema deverá ainda guardar o cadastro, mas não enviar notificação.
5. Teste `/api/cadastro/listar`, `/api/cadastro/estatisticas`, `/api/cadastro/:id`, `PUT` e `DELETE` com `Authorization: Bearer <ADMIN_API_TOKEN>`.
6. Teste que os endpoints administrativos recusam pedidos sem token.

## 7. Importação de uma base SQLite anterior (opcional)
O pacote organizado não inclui bases de dados com dados pessoais. Se tiver uma base SQLite anterior, mantenha-a fora do repositório e use o script `backend/scripts/import-sqlite.js` após configurar PostgreSQL e instalar as dependências de desenvolvimento (`npm install --omit=dev`). Defina `SQLITE_SOURCE` para o caminho do ficheiro SQLite se ele não estiver em `backend/ajov_cadastro.db`. Execute o script a partir da pasta `backend`:
```bash
node scripts/import-sqlite.js
```
O script evita duplicar IDs já existentes.

## 8. Desenvolvimento local
Na pasta `backend`, copie `.env.example` para `.env`, configure PostgreSQL e execute:
```bash
npm install
npm start
```
Sirva o frontend com um servidor estático na porta 5500 (ex.: `python -m http.server 5500`, executado dentro de `frontend`). O frontend detecta localhost e chama o backend na porta 3000.

## Limitações importantes
- A publicação completa depende de criar os serviços Netlify/Render, fornecer a URL real do backend e configurar PostgreSQL e credenciais. O ZIP não pode conter credenciais reais nem ser publicado sem esses valores.
- Antes de abrir inscrições em grande escala, considere CAPTCHA e rate limiting na rota pública de criação.
