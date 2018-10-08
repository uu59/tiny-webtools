---
title: 2018 Google News feed URL generator
layout: single
---


<form>
  <table>
  <tr>
    <th>Keyword</th>
    <td>
      <input name="q" type="text" placeholder="e.g. Amazon" />
    </td>
  </tr>
  <tr>
    <th>Region</th>
    <td>
      <select name="r">
        <option value="jp">JP</option>
        <option value="en_us">en_US</option>
      </select>
    </td>
  </tr>
  <tr>
    <th>Region Detail (optional)</th>
    <td>
      ned: <input name="ned" type="text" placeholder="e.g. jp" /><br>
      hl: <input name="hl" type="text" placeholder="e.g. ja" /><br>
      gl: <input name="gl" type="text" placeholder="e.g. JP" /><br>
    </td>
  </tr>
  </table>
  <input type="submit" value="generate" />
  <p>
    <a id="result"></a>
  </p>
  <div id="result-items"></div>
</form>

<script>
  const f = document.forms[0];
  const result = document.querySelector('#result')

  f.onsubmit = (ev) => {
    ev.preventDefault()
    const params = {}
    for(let el of ev.target.elements) {
      switch(el.name) {
        case "q":
          params.q = encodeURIComponent(el.value)
          break;

        case "r":
          switch(el.value.toLowerCase()) {
            case "jp":
              params.ned = "jp"
              params.hl = "ja"
              params.gl = "JP"
              break;
            case "en_us":
              params.ned = "us"
              params.hl = "en"
              params.gl = "US"
              break;
          }
          break;

        case "ned":
        case "hl":
        case "gl":
          if(el.value) {
            params[el.name] = el.value
          }
          break;
      }
    }
    if(!params.q) return;

    const url = `https://news.google.com/news/rss/headlines/section/q/${params.q}/?ned=${params.ned}&hl=${params.hl}&gl=${params.gl}`
    result.href = url;
    result.textContent = url;

    fetchContentPreview(url, decodeURIComponent(params.q))
  }

async function fetchContentPreview(url, q) {
  const resultItems = document.querySelector('#result-items')
  resultItems.innerHTML = `Loading preview for '${q}'..`

  const proxy = "https://cors-anywhere.herokuapp.com/"
  const response = await fetch(`${proxy}${url}`, {
    headers: {
      "x-requested-with": "xhr"
    }
  })
  const feed = await response.text()
  const parser = new DOMParser()
  const doc = parser.parseFromString(feed, "application/xml")

  const a = document.createElement('a')
  let html = "<ol>"
  for(let item of doc.querySelectorAll('item')) {
    let url = item.querySelector('link').textContent;
    a.href = url
    let domain = a.hostname
    let title = item.querySelector('title').textContent;
    let timestamp = new Date(item.querySelector('pubDate').textContent);
    html += `<li style="margin-bottom:1em">${timestamp.toLocaleString()} | ${domain}<br><a href="${url}">${title}</a></li>`
  }
  html += '</ol>'
  resultItems.innerHTML = html
}
</script>
