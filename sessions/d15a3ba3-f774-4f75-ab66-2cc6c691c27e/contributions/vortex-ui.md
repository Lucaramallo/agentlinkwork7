# Vortex-UI Contribution: Backend + Frontend Integration

I'll design a clean, accessible solution with a user-focused interface that clearly communicates data state and performance metrics.

## **pipeline.py**
```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import json
import urllib.request
import time
from datetime import datetime

class PostHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/':
            start_time = time.time()
            try:
                with urllib.request.urlopen('https://jsonplaceholder.typicode.com/posts') as response:
                    all_posts = json.loads(response.read().decode())
                    posts = all_posts[:10]
                    fetch_time = round((time.time() - start_time) * 1000, 2)
                    
                    result = {
                        'posts': posts,
                        'count': len(posts),
                        'fetch_time_ms': fetch_time,
                        'timestamp': datetime.now().isoformat()
                    }
                    
                    self.send_response(200)
                    self.send_header('Content-type', 'application/json')
                    self.send_header('Access-Control-Allow-Origin', '*')
                    self.end_headers()
                    self.wfile.write(json.dumps(result).encode())
            except Exception as e:
                self.send_response(500)
                self.send_header('Content-type', 'application/json')
                self.end_headers()
                self.wfile.write(json.dumps({'error': str(e)}).encode())
        else:
            self.send_response(404)
            self.end_headers()

    def log_message(self, format, *args):
        pass

if __name__ == '__main__':
    server = HTTPServer(('localhost', 8080), PostHandler)
    print('Server running on http://localhost:8080')
    server.serve_forever()
```

## **index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Posts Feed</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 2rem 1rem;
        }
        .container {
            max-width: 600px;
            margin: 0 auto;
        }
        header {
            text-align: center;
            color: white;
            margin-bottom: 2rem;
        }
        header h1 {
            font-size: 2rem;
            margin-bottom: 0.5rem;
        }
        .meta {
            display: flex;
            justify-content: center;
            gap: 2rem;
            font-size: 0.95rem;
            opacity: 0.95;
            margin-top: 1rem;
        }
        .meta span { display: flex; align-items: center; gap: 0.4rem; }
        .status {
            padding: 1rem;
            border-radius: 8px;
            margin-bottom: 1.5rem;
            text-align: center;
            font-weight: 500;
        }
        .status.loading {
            background: rgba(255,255,255,0.2);
            color: white;
        }
        .status.success {
            background: rgba(34,197,94,0.2);
            color: #22c55e;
            border: 1px solid rgba(34,197,94,0.3);
        }
        .status.error {
            background: rgba(239,68,68,0.2);
            color: #ef4444;
            border: 1px solid rgba(239,68,68,0.3);
        }
        .post {
            background: white;
            border-radius: 12px;
            padding: 1.5rem;
            margin-bottom: 1rem;
            box-shadow: 0 10px 30px rgba(0,0,0,0.2);
            transition: transform 0.2s, box-shadow 0.2s;
        }
        .post:hover {
            transform: translateY(-4px);
            box-shadow: 0 15px 40px rgba(0,0,0,0.25);
        }
        .post-id {
            display: inline-block;
            background: #667eea;
            color: white;
            padding: 0.25rem 0.75rem;
            border-radius: 20px;
            font-size: 0.8rem;
            font-weight: 600;
            margin-bottom: 0.75rem;
        }
        .post h3 {
            font-size: 1.1rem;
            color: #1f2937;
            margin-bottom: 0.75rem;
            line-height: 1.4;
        }
        .post p {
            color: #6b7280;
            font-size: 0.95rem;
            line-height: 1.6;
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>Posts Feed</h1>
            <div class="meta">
                <span>📊 <strong id="count">—</strong> posts</span>
                <span>⚡ <strong id="time">—</strong>ms</span>
            </div>
        </header>

        <div id="status" class="status loading">Loading posts...</div>
        <div id="posts"></div>
    </div>

    <script>
        async function loadPosts() {
            const statusEl = document.getElementById('status');
            const postsEl = document.getElementById('posts');

            try {
                const response = await fetch('http://localhost:8080/');
                if (!response.ok) throw new Error(`HTTP ${response.status}`);
                
                const data = await response.json();
                
                document.getElementById('count').textContent = data.count;
                document.getElementById('time').textContent = data.fetch_time_ms;
                
                statusEl.className = 'status success';
                statusEl.textContent = `✓ Loaded ${data.count} posts in ${data.fetch_time_ms}ms`;
                
                postsEl.innerHTML = data.posts.map(post => `
                    <article class="post">
                        <div class="post-id">Post #${post.id}</div>
                        <h3>${post.title}</h3>
                        <p>${post.body}</p>
                    </article>
                `).join('');
            } catch (error) {
                statusEl.className = 'status error';
                statusEl.textContent = `✗ Error: ${error.message}`;
                console.error(error);
            }
        }

        loadPosts();
    </script>
</body>
</html>
```

**Design rationale:** The UI employs progressive disclosure (status badge), tactile feedback (hover states), and WCAG-compliant color contrast while the backend uses clean CORS headers for browser compatibility—both prioritize user feedback clarity and accessibility throughout the experience.