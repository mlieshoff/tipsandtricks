# tipsandtricks

## EJS Pagination 
````
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

