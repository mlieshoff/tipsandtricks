# tipsandtricks

## Server Monitor 

````
#!/bin/bash

# ==== Konfiguration ====
URLS=(
  "https://example.com"
  "http://localhost:3000"
  "https://doesnotexist.local"
)

HOSTS=(
  "google.com:80"
  "localhost:8080"
  "192.168.1.1:22"
)

# ==== URL-Check ====
echo ">>> URL Reachability Check (via curl)"
for url in "${URLS[@]}"; do
  status=$(curl -s -o /dev/null -w "%{http_code}" "$url")
  if [ "$status" == "000" ]; then
    echo "❌ $url => NOT reachable"
  else
    echo "✅ $url => HTTP $status"
  fi
done

# ==== Host:Port-Check ====
echo ""
echo ">>> Host:Port Reachability Check (via nc)"
for hostport in "${HOSTS[@]}"; do
  IFS=':' read -r host port <<< "$hostport"
  nc -z -w2 "$host" "$port" &>/dev/null
  if [ $? -eq 0 ]; then
    echo "✅ $host:$port => reachable"
  else
    echo "❌ $host:$port => NOT reachable"
  fi
done
````

## EJS Pagination 
````
app.get('/messages', async (req, res) => {
  const perPage = 10;
  const page = parseInt(req.query.page) || 1;

  const [totalCount, messages] = await Promise.all([
    db.countMessages(), // z. B. SELECT COUNT(*) FROM messages
    db.getMessages(page, perPage) // SELECT * LIMIT x OFFSET y
  ]);

  const totalPages = Math.ceil(totalCount / perPage);

  res.render('pages/messages', {
    messages,
    currentPage: page,
    totalPages
  });
});

<ul class="divide-y border rounded">
  <% messages.forEach(msg => { %>
    <li class="p-4"><%= msg.content %></li>
  <% }) %>
</ul>

<!-- Pagination -->
<div class="flex justify-center mt-6 space-x-2">
  <% for (let i = 1; i <= totalPages; i++) { %>
    <a href="/messages?page=<%= i %>"
       class="px-4 py-2 border rounded 
              <%= currentPage === i ? 'bg-blue-600 text-white' : 'bg-white hover:bg-blue-100' %>">
      <%= i %>
    </a>
  <% } %>
</div>

<% if (totalPages > 1) { %>
  <nav class="flex justify-center mt-6 space-x-1 text-sm" aria-label="Pagination">

    <% if (currentPage > 1) { %>
      <a href="?page=<%= currentPage - 1 %>" class="px-3 py-2 border rounded hover:bg-gray-100">
        &laquo; Zurück
      </a>
    <% } %>

    <% for (let i = 1; i <= totalPages; i++) { %>
      <a href="?page=<%= i %>"
         class="px-3 py-2 border rounded
                <%= i === currentPage ? 'bg-blue-600 text-white' : 'hover:bg-gray-100' %>">
        <%= i %>
      </a>
    <% } %>

    <% if (currentPage < totalPages) { %>
      <a href="?page=<%= currentPage + 1 %>" class="px-3 py-2 border rounded hover:bg-gray-100">
        Weiter &raquo;
      </a>
    <% } %>

  </nav>
<% } %>

<%- include('../partials/pagination', { totalPages, currentPage }) %>

res.render('messages', {
  messages,
  currentPage: page,
  totalPages,
  baseQuery: `?search=${encodeURIComponent(req.query.search || '')}&`
});

<a href="<%= baseQuery %>page=<%= i %>">...</a>
````

Ajax

````
app.get('/reports/list', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const reports = await getReportsForPage(page); // z. B. DB-Zugriff

  res.render('partials/reportList', { reports });
});


<div id="reportContainer">
  <%- include('../partials/reportList', { reports }) %>
</div>

<nav>
  <button onclick="loadPage(2)">Seite 2</button>
</nav>

<script>
  function loadPage(page) {
    fetch(`/reports/list?page=${page}`)
      .then(res => res.text())
      .then(html => {
        document.getElementById('reportContainer').innerHTML = html;
      });
  }
</script>

<ul id="reportList" class="divide-y border rounded">
  <% reports.forEach(report => { %>
    <li class="p-4"><%= report.title %></li>
  <% }) %>
</ul>
````

# fastify openapi
````
// 1. Installiere die folgenden Pakete: // npm install fastify @fastify/swagger @fastify/swagger-ui @fastify/formbody fastify-openapi-glue zod // npm install --save-dev openapi-typescript-codegen

// 2. Beispielhafte OpenAPI-Datei (openapi.yaml)

/** openapi: 3.0.0 info: title: Message API version: 1.0.0 paths: /messages: post: summary: Neue Nachricht anlegen operationId: createMessage requestBody: required: true content: application/json: schema: $ref: '#/components/schemas/MessageCreateDto' responses: '201': description: Nachricht erstellt /messages: get: summary: Alle Nachrichten abrufen operationId: getMessages responses: '200': description: Liste der Nachrichten content: application/json: schema: type: array items: $ref: '#/components/schemas/MessageDto' components: schemas: MessageCreateDto: type: object required: - content properties: content: type: string MessageDto: type: object properties: id: type: string content: type: string */

// 3. Generiere DTOs und Router-Glue: // npx openapi-typescript-codegen --input openapi.yaml --output ./src/generated --client none

// 4. Fastify-Server mit openapi-glue

import Fastify from 'fastify' import swagger from '@fastify/swagger' import swaggerUI from '@fastify/swagger-ui' import formbody from '@fastify/formbody' import FastifyOpenapiGlue from 'fastify-openapi-glue' import { z } from 'zod' import { MessageCreateDto, MessageDto } from './generated/models'

const fastify = Fastify()

const SPEC = './openapi.yaml' const SERVICE = { async createMessage(req: any, reply: any) { const schema = z.object({ content: z.string() }) const validated = schema.parse(req.body) const message: MessageDto = { id: crypto.randomUUID(), content: validated.content } messages.push(message) reply.code(201).send() }, async getMessages(req: any, reply: any) { reply.send(messages) } }

const options = { specification: SPEC, service: SERVICE, noAdditional: true, strict: true }

let messages: MessageDto[] = []

await fastify.register(formbody) await fastify.register(swagger, { openapi: { info: { title: 'Message API', version: '1.0.0' } } }) await fastify.register(swaggerUI, { routePrefix: '/docs' }) await fastify.register(FastifyOpenapiGlue, options)

fastify.listen({ port: 3000 }, err => { if (err) throw err console.log('Server bereit unter http://localhost:3000') })

````
Hier ist eine kompakte Übersicht, wie du Fastify-Endpunkte automatisch aus einer OpenAPI-Datei generieren und anbinden kannst – inklusive DTOs und Glue-Code:


---

🧱 Komponenten

Baustein	Zweck

openapi.yaml	Deine API-Definition (mit operationId)
openapi-typescript-codegen	Generiert DTOs aus dem OpenAPI-Schema
fastify-openapi-glue	Verbindet Endpunkte dynamisch an deine Handlers



---

🔧 Setup Schritt für Schritt

1. 📄 openapi.yaml definieren

paths:
  /messages:
    get:
      summary: Alle Nachrichten
      operationId: getMessages
      responses:
        '200':
          description: Liste

> Wichtig: operationId ist entscheidend! Es wird zur Methodenzuordnung verwendet.




---

2. 🛠️ DTOs generieren (optional aber empfohlen)

npx openapi-typescript-codegen \
  --input openapi.yaml \
  --output src/generated \
  --client none

Ergebnis: src/generated/models/MessageDto.ts usw.


---

3. 📦 fastify-openapi-glue installieren

npm install fastify-openapi-glue


---

4. 🧩 Glue verwenden im Fastify-Server

import Fastify from 'fastify'
import FastifyOpenapiGlue from 'fastify-openapi-glue'

const fastify = Fastify()

const SERVICE = {
  async getMessages(req, reply) {
    reply.send([{ id: '1', content: 'Hallo' }])
  }
}

const options = {
  specification: './openapi.yaml',
  service: SERVICE,
  noAdditional: true,
  strict: true
}

await fastify.register(FastifyOpenapiGlue, options)


---

5. 🧪 Testen

curl http://localhost:3000/messages


---

🧠 Hinweise

Du brauchst operationId für jede Route in OpenAPI.

Dein service-Objekt muss exakt diese Methoden bereitstellen.

Die Validation übernimmt fastify-openapi-glue automatisch basierend auf dem Schema.



---

Wenn du willst, kann ich dir eine funktionierende Minimal-Repo-Struktur erstellen oder das Ganze mit Swagger-UI, Zod oder Prisma erweitern. Sag einfach Bescheid! ✅

Sehr gut – hier ist, wie du aus deiner openapi.yaml automatisch einen TypeScript-Client generierst, der perfekt zu deinem Fastify-Backend passt und direkt nutzbar ist (z. B. mit axios oder fetch):


---

🧰 Tool: openapi-typescript-codegen

📦 Installation:

npm install --save-dev openapi-typescript-codegen


---

🔧 Client generieren:

npx openapi-typescript-codegen \
  --input openapi.yaml \
  --output ./src/client \
  --client axios

> Alternativen zu --client axios: fetch oder node.




---

🗂️ Ergebnisstruktur:

src/client/
├── core/
│   ├── ApiError.ts
│   ├── request.ts
│   └── ...
├── models/
│   └── MessageDto.ts
├── services/
│   └── MessageService.ts
└── index.ts


---

✅ Beispielverwendung (Axios)

import { MessageService } from './client'

async function loadMessages() {
  const messages = await MessageService.getMessages()
  console.log(messages)
}

> Auth, Header oder BaseURL kannst du beim Import konfigurieren – z. B. in client/core/ApiConfig.ts.




---

🛠️ Optional: package.json Script

{
  "scripts": {
    "generate-client": "openapi-typescript-codegen --input openapi.yaml --output ./src/client --client axios"
  }
}


---

💡 Warum das nützlich ist:

Backend & Frontend bleiben synchron

DTOs und Endpunkte sind garantiert kompatibel

Keine Copy-Paste-Typen mehr

Schnell nutzbar in React, Vue, Svelte, usw.



---

Möchtest du auch:

Authentifizierung automatisch im Client injecten?

vite oder next.js direkt konfigurieren?

die Backend-Daten validieren lassen (z. B. Zod)?


Sag einfach: „Client bitte mit Auth + Vite“ oder was du brauchst 💪

Perfekt – hier zeige ich dir, wie du Authentifizierung in den generierten TypeScript-Client injecten kannst, z. B. für Bearer-Tokens oder API-Keys.


---

🔐 Ziel: Automatisch JWT oder API-Key mitschicken bei jedem Request


---

🧱 Ausgangslage

Du hast bereits generiert mit:

npx openapi-typescript-codegen \
  --input openapi.yaml \
  --output ./src/client \
  --client axios


---

✅ Lösung: request.config.ts anpassen

Geh in:

src/client/core/request.ts

und ergänze dynamisch injizierbare Auth-Header wie folgt:


---

🛠️ Bearer-Token einfügen

import axios from 'axios'
import { getToken } from './auth'

export const request = async (config) => {
  const token = getToken() // Funktion, die dein Token aus Speicher / Cookie / Store holt

  return axios.request({
    ...config,
    headers: {
      ...(config.headers || {}),
      Authorization: token ? `Bearer ${token}` : '',
    },
  })
}


---

🔁 Beispiel auth.ts

export function getToken(): string | null {
  // z. B. aus localStorage oder globaler Store
  return localStorage.getItem('jwt_token')
}


---

💡 Alternativen:

Wenn du API-Key brauchst:

headers: {
  'X-API-Key': getApiKey()
}

Wenn du Token refreshen willst, kannst du axios.interceptors verwenden.



---

🚀 Jetzt in der Anwendung

import { MessageService } from '../client'

await MessageService.getMessages()  // JWT wird automatisch angehängt


---

🧠 Bonus-Tipp: Typ-sicheres Login

Wenn dein OpenAPI-File ein Login-Endpoint hat (z. B. /auth/login), wird automatisch ein AuthService.login(...) generiert – nutz das direkt für deine Login-Logik + Token-Storage.


---

Möchtest du ein komplettes Beispielprojekt mit:

Login-Seite (React oder EJS)

Fastify-Backend mit JWT-Signierung

Client mit Token-Handling


Sag einfach: „Bitte Komplettprojekt JWT“ ✅




# sed credentials replacement

Hier ist eine einfache und effektive sed-Lösung, um Credentials oder Platzhalterwerte in deinen .env-, docker-compose.yml- oder Config-Dateien beim Deployment zu ersetzen:


---

🔧 Beispiel: .env.template

DB_USER=__DB_USER__
DB_PASS=__DB_PASS__


---

✅ sed-Befehl (Linux/macOS kompatibel)

sed \
  -e "s|__DB_USER__|meinuser|g" \
  -e "s|__DB_PASS__|geheim123|g" \
  .env.template > .env

🔒 Die generierte .env enthält dann echte Werte:

DB_USER=meinuser
DB_PASS=geheim123


---

💡 Für docker-compose.yml geht es genauso

sed \
  -e "s|__JWT_SECRET__|$(cat .secrets/jwt.txt)|g" \
  -e "s|__API_KEY__|$MY_API_KEY|g" \
  docker-compose.template.yml > docker-compose.yml


---

📁 Empfehlung für Struktur

infra/
├── docker-compose.template.yml
├── .env.template
├── deploy.sh         # führt das sed-Replacement durch
└── .secrets/
    ├── jwt.txt
    └── pg-password.txt


---

🧪 Sicherheitstipps

.env und docker-compose.yml nicht ins Git committen

Replaced-Dateien ggf. am Ende löschen (rm .env)

Skript nur auf vertrauenswürdigen Systemen verwenden



---

Hier ist ein fertiges, sicheres und leicht anpassbares Beispiel für dein deploy.sh-Script – ideal für deine Podman-/Docker-Infrastruktur mit .env- und docker-compose-Templates:


---

✅ deploy.sh

#!/bin/bash

set -e  # bricht bei Fehler ab

echo "🔧 Starte Deployment..."

# Pfade
TEMPLATE_ENV=".env.template"
TARGET_ENV=".env"
TEMPLATE_COMPOSE="docker-compose.template.yml"
TARGET_COMPOSE="docker-compose.yml"
SECRETS_DIR="./.secrets"

# Ersetze Platzhalter in .env
echo "📄 Ersetze Secrets in .env..."
sed \
  -e "s|__DB_USER__|$(cat $SECRETS_DIR/db_user.txt)|g" \
  -e "s|__DB_PASS__|$(cat $SECRETS_DIR/db_pass.txt)|g" \
  $TEMPLATE_ENV > $TARGET_ENV

# Ersetze Platzhalter in docker-compose.yml
echo "📄 Ersetze Secrets in docker-compose.yml..."
sed \
  -e "s|__JWT_SECRET__|$(cat $SECRETS_DIR/jwt_secret.txt)|g" \
  $TEMPLATE_COMPOSE > $TARGET_COMPOSE

# Deployment starten
echo "🚀 Starte podman-compose..."
podman-compose up -d

echo "✅ Deployment abgeschlossen!"


---

📁 Struktur dazu

infra/
├── deploy.sh
├── docker-compose.template.yml
├── .env.template
├── .secrets/
│   ├── db_user.txt
│   ├── db_pass.txt
│   └── jwt_secret.txt


---

🔐 Sicherheit

.secrets/ in .gitignore aufnehmen

chmod +x deploy.sh nicht vergessen

ggf. .env und docker-compose.yml nach Deployment löschen (optional in Script ergänzbar)


---
Hier ist die erweiterte, robuste Version von deploy.sh, inklusive:

✅ Validierung der Secrets
✅ Backup von alten .env- und Compose-Dateien
✅ Farbausgabe
✅ Fehlerausgabe
✅ Automatischer Cleanup optional


---

💻 deploy.sh

#!/bin/bash

set -euo pipefail

### Farben
GREEN='\033[0;32m'
RED='\033[0;31m'
CYAN='\033[0;36m'
NC='\033[0m' # No Color

info()    { echo -e "${CYAN}ℹ️  $1${NC}"; }
success() { echo -e "${GREEN}✅ $1${NC}"; }
error()   { echo -e "${RED}❌ $1${NC}" >&2; }

### Dateien und Pfade
TEMPLATE_ENV=".env.template"
TARGET_ENV=".env"
TEMPLATE_COMPOSE="docker-compose.template.yml"
TARGET_COMPOSE="docker-compose.yml"
SECRETS_DIR="./.secrets"
BACKUP_DIR="./.backup"

REQUIRED_SECRETS=("db_user.txt" "db_pass.txt" "jwt_secret.txt")

### Vorbereitung
info "Prüfe erforderliche Dateien..."
mkdir -p "$BACKUP_DIR"

for secret in "${REQUIRED_SECRETS[@]}"; do
  if [[ ! -f "$SECRETS_DIR/$secret" ]]; then
    error "Fehlende Secret-Datei: $SECRETS_DIR/$secret"
    exit 1
  fi
done

[[ -f "$TARGET_ENV" ]] && cp "$TARGET_ENV" "$BACKUP_DIR/env.bak"
[[ -f "$TARGET_COMPOSE" ]] && cp "$TARGET_COMPOSE" "$BACKUP_DIR/compose.bak"

### Ersetze .env
info "Ersetze Secrets in .env..."
sed \
  -e "s|__DB_USER__|$(< $SECRETS_DIR/db_user.txt)|g" \
  -e "s|__DB_PASS__|$(< $SECRETS_DIR/db_pass.txt)|g" \
  "$TEMPLATE_ENV" > "$TARGET_ENV"

### Ersetze docker-compose.yml
info "Ersetze Secrets in docker-compose.yml..."
sed \
  -e "s|__JWT_SECRET__|$(< $SECRETS_DIR/jwt_secret.txt)|g" \
  "$TEMPLATE_COMPOSE" > "$TARGET_COMPOSE"

### Starte Deployment
info "Starte podman-compose..."
if podman-compose up -d; then
  success "Deployment erfolgreich abgeschlossen."
else
  error "Deployment fehlgeschlagen."
  exit 1
fi

### Optional: Cleanup generierte Dateien nach Start
# rm "$TARGET_ENV" "$TARGET_COMPOSE"
# info ".env und docker-compose.yml wurden entfernt (Cleanup aktiviert)."

exit 0


---

📁 Empfohlene Struktur

infra/
├── deploy.sh
├── .env.template
├── docker-compose.template.yml
├── .backup/
├── .secrets/
│   ├── db_user.txt
│   ├── db_pass.txt
│   └── jwt_secret.txt

----
# podman traefik 

Super! Hier kommt ein funktionierendes Minimalbeispiel für Traefik mit Podman rootless, inklusive:

traefik als Reverse Proxy

whoami als Testservice

HTTPS via selbstsigniertem Zertifikat

Alles in einem rootless-kompatiblen Setup



---

📁 Verzeichnisstruktur

projekt/
├── docker-compose.yml
├── traefik.yml
├── certs/
│   ├── local.crt
│   └── local.key


---

📝 1. traefik.yml

api:
  dashboard: true
  insecure: true

entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"

tls:
  certificates:
    - certFile: "/certs/local.crt"
      keyFile: "/certs/local.key"

providers:
  docker: {}


---

📝 2. docker-compose.yml

version: "3.9"

services:
  traefik:
    image: traefik:v3.0
    container_name: traefik
    command:
      - "--providers.docker=true"
      - "--api.dashboard=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
    ports:
      - "80:80"
      - "443:443"
      - "9000:8080" # Dashboard über http://localhost:9000
    volumes:
      - "./traefik.yml:/etc/traefik/traefik.yml"
      - "./certs:/certs:ro"
      - "$XDG_RUNTIME_DIR/podman/podman.sock:/var/run/docker.sock:ro"
    networks:
      - web

  whoami:
    image: traefik/whoami
    container_name: whoami
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`localhost`)"
      - "traefik.http.routers.whoami.entrypoints=websecure"
      - "traefik.http.routers.whoami.tls=true"
    networks:
      - web

networks:
  web:
    driver: bridge


---

🛠️ 3. Zertifikate erstellen

mkdir -p certs

openssl req -x509 -newkey rsa:2048 -nodes \
  -keyout certs/local.key \
  -out certs/local.crt \
  -days 365 \
  -subj "/CN=localhost"


---

▶️ 4. Starten

podman-compose down -v
podman-compose up -d


---

✅ Test

🔒 https://localhost → sollte whoami anzeigen

🧰 http://localhost:9000/dashboard/ → Dashboard


> Hinweis: Dein Browser wird eine Warnung wegen des selbstsignierten Zertifikats anzeigen – akzeptieren.




---

❗️ Wichtig für Rootless Podman

Falls podman-compose nicht automatisch podman.socket startet:

systemctl --user start podman.socket
loginctl enable-linger $USER

Oder explizit:

export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/podman/podman.sock


---

Wenn du möchtest, erweitere ich das Beispiel gerne mit deinem Auth-Service, Metabase, RabbitMQ oder sonstigen Komponenten. Sag einfach Bescheid!

---


Sehr gerne! Hier bekommst du ein komplett lauffähiges Podman-rootless-Traefik-Setup mit providers.file statt Docker-Labels. Das bedeutet:

Du konfigurierst alle Routen in einer eigenen YAML-Datei

Kein Zugriff auf docker.sock nötig → perfekt für rootless

TLS über self-signed Zertifikat



---

📁 Verzeichnisstruktur

projekt/
├── docker-compose.yml
├── traefik.yml              # Traefik-Konfiguration
├── routes.yml               # File-based Routing
├── certs/
│   ├── local.crt
│   └── local.key


---

📝 traefik.yml

api:
  dashboard: true
  insecure: true

entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"

tls:
  certificates:
    - certFile: "/certs/local.crt"
      keyFile: "/certs/local.key"

providers:
  file:
    filename: /etc/traefik/routes.yml
    watch: true


---

📝 routes.yml

http:
  routers:
    whoami-router:
      rule: "Host(`localhost`)"
      entryPoints:
        - websecure
      service: whoami-service
      tls: true

  services:
    whoami-service:
      loadBalancer:
        servers:
          - url: "http://whoami:80"


---

📝 docker-compose.yml

version: "3.9"

services:
  traefik:
    image: traefik:v3.0
    container_name: traefik
    command:
      - "--providers.file.filename=/etc/traefik/routes.yml"
      - "--api.dashboard=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
    ports:
      - "80:80"
      - "443:443"
      - "9000:8080"
    volumes:
      - "./traefik.yml:/etc/traefik/traefik.yml"
      - "./routes.yml:/etc/traefik/routes.yml"
      - "./certs:/certs:ro"
    networks:
      - web

  whoami:
    image: traefik/whoami
    container_name: whoami
    networks:
      - web

networks:
  web:
    driver: bridge


---

🛠️ Zertifikat erstellen

Falls noch nicht vorhanden:

mkdir -p certs

openssl req -x509 -newkey rsa:2048 -nodes \
  -keyout certs/local.key \
  -out certs/local.crt \
  -days 365 \
  -subj "/CN=localhost"


---

▶️ Starten

podman-compose down -v
podman-compose up -d


---

✅ Test

➡️ https://localhost → zeigt die whoami-Antwort

➡️ http://localhost:9000/dashboard/ → zeigt das Traefik-Dashboard


> Hinweis: Bei self-signed-Zertifikat gibt dein Browser eine Warnung – das ist normal.




---

Wenn du möchtest, erweitere ich es dir auch gleich mit auth, rabbitmq, oder einer dynamischen .env-basierten Routensteuerung. Sag einfach Bescheid: "Mach mir ein Auth-Service Beispiel" oder ähnliches.

