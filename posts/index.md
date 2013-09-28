  <article>

<h1>Rob's Blog</h1>

<p>These posts reflect my own personal opinions and thoughts. Here are some things I work on:

<ul style="list-style:circle;">
  <li>[Kal](http://rzimmerman.github.io/kal)</li>
  <li>[Curiosity](http://www.nasa.gov/mission_pages/msl/index.html)</li>
</ul>
</p>

<p>I mostly write about my personal work here. Feel free to contact me with corrections or comments.</p>

<p>Like the way this blog looks? You have strange taste, but feel free to fork/steal it on <a href="https://www.github.com/rzimmerman/roblog">Github</a>.</p>

</article>

  <%
    _.each(posts.reverse(), function(file) {
      var post = postName(file);
      var data = manifest[file];
      if (post == 'index') return;
  %>
  <article class="excerpt">
  <h3><a href="/<%= post %>/"><%= data.title %></a></h3>
  <p><%= data.description%></p>
  </article>
  <% }); %>

