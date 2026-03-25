<!DOCTYPE html>
<html>
<head>
  <title>YouTube Search Tool</title>
  <style>
    body { font-family: Arial; padding: 20px; }
    input, button, select { padding: 8px; margin: 5px; }
    .result { margin: 10px 0; }
  </style>
</head>
<body>

<h2>YouTube Search</h2>

<select id="mode">
  <option value="video">Search Videos</option>
  <option value="channel">Search Users</option>
</select>

<input type="text" id="query" placeholder="Enter search..." />
<button onclick="search()">Search</button>

<div id="results"></div>

<script>
const API_KEY = "YOUR_API_KEY";

async function search() {
  const mode = document.getElementById("mode").value;
  const query = document.getElementById("query").value;
  const resultsDiv = document.getElementById("results");
  resultsDiv.innerHTML = "Loading...";

  if (mode === "video") {
    const res = await fetch(
      `https://www.googleapis.com/youtube/v3/search?part=snippet&q=${query}&type=video&maxResults=10&key=${API_KEY}`
    );
    const data = await res.json();

    resultsDiv.innerHTML = "";
    data.items.forEach(item => {
      const videoId = item.id.videoId;
      const title = item.snippet.title;
      const channel = item.snippet.channelTitle;
      const url = `https://www.youtube.com/watch?v=${videoId}`;

      resultsDiv.innerHTML += `
        <div class="result">
          <b>${title}</b><br>
          Channel: ${channel}<br>
          <a href="${url}" target="_blank">${url}</a>
        </div>
      `;
    });

  } else {
    const res = await fetch(
      `https://www.googleapis.com/youtube/v3/search?part=snippet&q=${query}&type=channel&maxResults=20&key=${API_KEY}`
    );
    const data = await res.json();

    resultsDiv.innerHTML = "<h3>Select a channel:</h3>";

    data.items.forEach(item => {
      const channelId = item.id.channelId;
      const name = item.snippet.title;

      resultsDiv.innerHTML += `
        <div class="result">
          <button onclick="loadChannel('${channelId}')">${name}</button>
        </div>
      `;
    });
  }
}

async function loadChannel(channelId) {
  const resultsDiv = document.getElementById("results");
  resultsDiv.innerHTML = "Loading videos...";

  const res = await fetch(
    `https://www.googleapis.com/youtube/v3/search?part=snippet&channelId=${channelId}&order=date&type=video&maxResults=10&key=${API_KEY}`
  );
  const data = await res.json();

  resultsDiv.innerHTML = "<h3>Recent Videos:</h3>";

  data.items.forEach(item => {
    const videoId = item.id.videoId;
    const title = item.snippet.title;
    const url = `https://www.youtube.com/watch?v=${videoId}`;

    resultsDiv.innerHTML += `
      <div class="result">
        <b>${title}</b><br>
        <a href="${url}" target="_blank">${url}</a>
      </div>
    `;
  });
}
</script>

</body>
</html>
