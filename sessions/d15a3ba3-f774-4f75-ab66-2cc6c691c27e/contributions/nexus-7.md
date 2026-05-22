**Problem:** Need a decoupled backend-frontend system with CORS support and client-side performance metrics for API consumption.

**Solution:** Build a Flask server (pipeline.py) exposing filtered posts via `/posts` endpoint with CORS headers, and a vanilla HTML/JS client (index.html) that fetches asynchronously, renders cards, and logs performance timing via `performance.now()`.

**Implementation:**

**pipeline.py:**
```python
from flask import Flask, jsonify
from flask_cors import CORS
import requests

app = Flask(__name__)
CORS(app)

@app.route('/posts', methods=['GET'])
def get_posts():
    response = requests.get('https://jsonplaceholder.typicode.com/posts')
    posts = response.json()[:10]
    return jsonify(posts)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

**index.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Posts Feed</title>
    <style>
        body { font-family: Arial; max-width: 800px; margin: 20px auto; }
        .stats { background: #f0f0f0; padding: 10px; margin-bottom: 20px; border-radius: 4px; }
        .card { border: 1px solid #ddd; padding: 15px; margin: 10px 0; border-radius: 4px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .card h3 { margin-top: 0; }
    </style>
</head>
<body>
    <h1>Posts Feed</h1>
    <div class="stats"><p>Records: <span id="count">0</span> | Fetch Time: <span id="time">0</span>ms</p></div>
    <div id="posts"></div>
    <script>
        const start = performance.now();
        fetch('http://localhost:8080/posts')
            .then(r => r.json())
            .then(data => {
                document.getElementById('count').textContent = data.length;
                document.getElementById('time').textContent = (performance.now() - start).toFixed(2);
                document.getElementById('posts').innerHTML = data.map(p => 
                    `<div class="card"><h3>${p.title}</h3><p>${p.body}</p></div>`
                ).join('');
            });
    </script>
</body>
</html>
```

**Execution:** Install dependencies (`pip install flask flask-cors requests`), run `python3 pipeline.py`, open `index.html` in browser—CORS enabled, zero external dependencies in frontend.