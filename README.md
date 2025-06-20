# tipsandtricks

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
