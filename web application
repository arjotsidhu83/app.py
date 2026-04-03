from flask import Flask, request, jsonify, render_template_string
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy import func

app = Flask(__name__)

# 1. DATABASE CONFIGURATION
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///feedback.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

class Feedback(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    email = db.Column(db.String(100), nullable=False)
    rating = db.Column(db.Integer, nullable=False)
    comments = db.Column(db.Text)

with app.app_context():
    db.create_all()

# 2. HTML TEMPLATES (Embedded as Strings)
HTML_LAYOUT = """
<!DOCTYPE html>
<html>
<head>
    <title>Feedback System</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #f0f2f5; display: flex; flex-direction: column; align-items: center; padding: 50px; }
        .card { background: white; padding: 2rem; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); width: 100%; max-width: 500px; }
        input, select, textarea { width: 100%; padding: 10px; margin: 10px 0; border: 1px solid #ddd; border-radius: 6px; box-sizing: border-box; }
        button { width: 100%; padding: 12px; background: #007bff; color: white; border: none; border-radius: 6px; cursor: pointer; font-size: 16px; }
        button:hover { background: #0056b3; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; background: white; }
        th, td { padding: 12px; border: 1px solid #eee; text-align: left; }
        th { background: #007bff; color: white; }
        .nav { margin-bottom: 20px; }
    </style>
</head>
<body>
    <div class="nav">
        <a href="/">Submit Feedback</a> | <a href="/admin">Admin Dashboard</a>
    </div>
    {% block content %}{% endblock %}
</body>
</html>
"""

INDEX_HTML = """
{% extends "layout" %}
{% block content %}
<div class="card">
    <h2>User Feedback Form</h2>
    <form id="fbForm">
        <input type="text" id="name" placeholder="Full Name" required>
        <input type="email" id="email" placeholder="Email Address" required>
        <select id="rating">
            <option value="5">5 - Excellent</option>
            <option value="4">4 - Good</option>
            <option value="3">3 - Average</option>
            <option value="2">2 - Poor</option>
            <option value="1">1 - Terrible</option>
        </select>
        <textarea id="comments" placeholder="Comments..." rows="4"></textarea>
        <button type="submit">Submit Feedback</button>
    </form>
    <p id="msg"></p>
</div>

<script>
document.getElementById('fbForm').addEventListener('submit', async (e) => {
    e.preventDefault();
    const data = {
        name: document.getElementById('name').value,
        email: document.getElementById('email').value,
        rating: document.getElementById('rating').value,
        comments: document.getElementById('comments').value
    };
    const res = await fetch('/api/submit', {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify(data)
    });
    if (res.ok) {
        document.getElementById('msg').innerText = "✅ Success! Thank you.";
        document.getElementById('fbForm').reset();
    }
});
</script>
{% endblock %}
"""

ADMIN_HTML = """
{% extends "layout" %}
{% block content %}
<div style="width: 80%;">
    <h2>Admin Dashboard</h2>
    <div class="card" style="margin-bottom: 20px;">
        <strong>Total Responses:</strong> {{ total }} <br>
        <strong>Average Rating:</strong> {{ avg }} / 5
    </div>
    <table>
        <tr><th>Name</th><th>Email</th><th>Rating</th><th>Comments</th></tr>
        {% for f in feedbacks %}
        <tr>
            <td>{{ f.name }}</td>
            <td>{{ f.email }}</td>
            <td>{{ f.rating }}</td>
            <td>{{ f.comments }}</td>
        </tr>
        {% endfor %}
    </table>
</div>
{% endblock %}
"""

# 3. ROUTES
@app.route('/')
def index():
    return render_template_string(INDEX_HTML)

@app.route('/layout') # Helper for template inheritance
def layout():
    return render_template_string(HTML_LAYOUT)

@app.route('/admin')
def admin():
    feedbacks = Feedback.query.all()
    avg = db.session.query(func.avg(Feedback.rating)).scalar() or 0
    total = Feedback.query.count()
    return render_template_string(ADMIN_HTML, feedbacks=feedbacks, avg=round(avg, 1), total=total)

@app.route('/api/submit', methods=['POST'])
def api_submit():
    data = request.json
    new_entry = Feedback(
        name=data['name'], 
        email=data['email'], 
        rating=int(data['rating']), 
        comments=data['comments']
    )
    db.session.add(new_entry)
    db.session.commit()
    return jsonify({"status": "success"})

if __name__ == '__main__':
    app.run(debug=True)
