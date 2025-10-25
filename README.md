# ⚡ VoltzV — Gestão de Vulnerabilidades (On‑Prem)
VoltzV é uma plataforma web on‑prem para centralizar, visualizar e gerenciar vulnerabilidades, ativos e integrações com scanners (ex.: Tenable). O projeto inclui:
- Frontend em PHP (`voltzv/`) com Bootstrap 5 + Chart.js + Tribute.js
- API principal em Node/Express + MongoDB (`root/api-node/`)
- License Manager em Node/Express + MongoDB (`license-manager/`)
- Modo Free (limite de 10 hosts) e licenças Pro/Pro+ com validação multi‑tenant
## 🧭 Arquitetura
- `voltzv/` (Frontend PHP)
  - Consome a API principal (JWT)
  - Persistência de licença no servidor local via `api/license_state.php`
  - Validação de licença contra servidor público (default `http://217.196.63.76`)
- `root/api-node/` (API Core)
  - Node.js + Express + Mongoose
  - Endpoints: `/login`, `/me`, `/users`, `/groups`, `/vulnerabilities`, `/dashboard/*`, `/analytics/*`, `/tenable/*`, `/import/*`, `/install/*`, `/certificates`, etc.
- `license-manager/` (Serviço de Licenças)
  - Node.js + Express + MongoDB (db `licenseDB`)
  - Rotas: `/clients`, `/licenses`, `/licenses/verify-license`
  - Suporte a `client_id` (tenant) e flag `used`
## 🔐 Licenciamento e Tenant
- Tipos: `free`, `pro`, `proplus`
- Licença é validada contra servidor público (`LICENSE_PUBLIC_BASE`, default `http://217.196.63.76`)
- Multi‑tenant:
  - Cada instalação define o `client_id` (tenant) localmente.
  - A ativação só é concluída se a licença informada pertencer ao tenant do cliente.
- Persistência:
  - Estado da licença é salvo no servidor local (`voltzv/api/license_state.php` → `voltzv/storage/license_state.json`).
  - Ao expirar, o plano volta automaticamente para `free`.
## 🚀 Quickstart (Docker)
1) Somente Frontend + License Manager (use uma API existente)
- Compose em `voltzv/deploy/docker-compose.yml`.
- Variáveis:
  - `API_BASE`: URL da API principal (ex.: `http://api.suaempresa:3000`)
  - `LICENSE_MANAGER_BASE`: URL do License Manager (ou use o proxy `voltzv/api/license.php`)
  - `LM_MONGO_URI`, `LM_PORT` (para o License Manager)
Exemplo:
```bash
cd voltzv/deploy
API_BASE=http://api:3000 \
LICENSE_MANAGER_BASE=http://license-manager:3000 \
docker compose up -d --build
```
Frontend: `http://localhost:8080/voltzv/`
2) Stack limpa (Frontend + API + Mongo dedicados)
- Sobe um Mongo “clean” (27018), uma API “clean” (3001) e um Web “clean” (8081), isolados da sua instalação atual.
- Script sugerido: `/home/kenayxkk/start-voltzv-stack-clean.sh`
Exemplo uso:
```bash
# Defaults: FE_PORT=8081 API_PORT=3001 MONGO_PORT=27018 DB_NAME=voltzv_clean
/home/kenayxkk/start-voltzv-stack-clean.sh
# ou customizando
FE_PORT=8082 API_PORT=3002 MONGO_PORT=27019 \
TENANT_CLIENT_ID=SEU_TENANT \
LICENSE_PUBLIC_BASE=http://217.196.63.76 \
/home/kenayxkk/start-voltzv-stack-clean.sh
```
URLs:
- Frontend: `http://localhost:8081/voltzv/`
- API: `http://localhost:3001`
- Mongo: `mongodb://localhost:27018/voltzv_clean`
## ⚙️ Configuração por ambiente
Frontend (`voltzv/`):
- `API_BASE`: base da API principal (injeção via `js/runtime-config.js.php`)
- `LICENSE_PUBLIC_BASE`: servidor público para validar licenças (default `http://217.196.63.76`)
- `TENANT_CLIENT_ID`: opcional; define o tenant ID da instalação
- `ALLOW_TENANT_UPDATE`: se definido, permite alterar o tenant via UI após setado
API (`root/api-node/`):
- `MONGO_URI` (ex.: `mongodb://localhost:27017/voltzv`)
- `PORT` (ex.: `3000`)
- `JWT_SECRET` (obrigatório)
License Manager (`license-manager/`):
- `MONGO_URI` (ex.: `mongodb://localhost:27017`)
- `PORT` (ex.: `3000`)
## 🧩 Principais Módulos (Frontend)
- Dashboard: cards + gráficos (severidade, status, fontes) e menções
- Vulnerabilidades: filtros, workflow de status, grupos e comentários com menções (@ usuário, ! grupo)
- Ativos: filtros, paginação; banner do Free apenas quando o plano é `free`
- Scans: listagem e filtros por fonte (Tenable, etc.)
- Catálogos: CVEs e Plugins Tenable
- Topologia: grafo de ativos (Vis.js)
- Heatmap: top ativos por quantidade de vulnerabilidades
- Gestão: certificados (CRUD, alerta de expiração, export CSV)
- Ações em Massa: transição de status, import de scans
- Configurações: MFA, integrações e aba “Licença” com ativação por tenant
- Licença (Config): define tenant (ID) e ativa licença Pro/Pro+ (validação servidor público)
- Logs: abas para usuários, integrações e sistema + export CSV
## 🔌 Integrações
- Tenable.VM: import `.nessus`, catálogo de plugins, analytics
- Placeholders para Rapid7/Qualys/Burp (telas e rotas preparadas para expansão)
## 📂 Estrutura de pastas (resumo)
```
voltzv/                 # Frontend PHP
  api/license.php       # Proxy para License Manager
  api/license_state.php # Estado/validação de licença (server-side)
  deploy/               # Dockerfile + compose do Web
  js/                   # app.js, license.js, runtime-config
  pages/                # telas: assets, vulnerabilities, scans, config, etc.
  storage/              # estado da licença (JSON) – criado em runtime
license-manager/        # Serviço de licenças (Node + Mongo)
  routes/               # clients.js, licenses.js
  models/               # Client.js, License.js
  server.js             # Express (porta 3000)
root/api-node/          # API principal (Node + Mongo)
  models/, routes/, ... # Endpoints e modelos principais
```
## 🛠 Desenvolvimento
Requisitos:
- Node.js 18+
- MongoDB 6+
- PHP 8.2 + Apache/Nginx (modo dev: `php -S` funciona)
Exemplo local (API + Web sem Docker):
```bash
# API
cd root/api-node
export MONGO_URI=mongodb://localhost:27017/voltzv
export PORT=3000
export JWT_SECRET=change_this_secret
node index.js
# Web (PHP embutido)
cd voltzv
API_BASE=http://localhost:3000 php -S 0.0.0.0:8080 -t .
# Acesse http://localhost:8080/voltzv/
```
## 🔒 Segurança e notas
- Evite IPs hardcoded; use `API_BASE` e `LICENSE_PUBLIC_BASE` por ambiente
- O plano não fica no navegador; é validado e persistido no servidor local
- Licença é multi‑tenant: só ativa se o código pertencer ao tenant da instalação
- Ao expirar, volta para `free` automaticamente
## 📃 Licença
Projeto interno/on‑prem. Ajuste este bloco conforme a política da sua organização.
