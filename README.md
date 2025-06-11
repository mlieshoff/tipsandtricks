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
