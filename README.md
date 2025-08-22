my_flask_app/
│
├── app.FOR
│   └──
        import sqlite3
        from flask import Flask, render_template, request, g, redirect, url_for

        app = Flask(__name__)
        DATABASE = "messages.db"

        def get_db():
            db = getattr(g, "_database", None)
            if db is None:
                db = g._database = sqlite3.connect(DATABASE)
                db.row_factory = sqlite3.Row
            return db

        @app.teardown_appcontext
        def close_connection(exception):
            db = getattr(g, "_database", None)
            if db is not None:
                db.close()

        def init_db():
            with app.app_context():
                db = get_db()
                cursor = db.cursor()
                cursor.execute("""
                    CREATE TABLE IF NOT EXISTS messages (
                        id INTEGER PRIMARY KEY AUTOINCREMENT,
                        name TEXT NOT NULL,
                        email TEXT NOT NULL,
                        message TEXT NOT NULL
                    )
                """)
                db.commit()

        @app.route("/")
        def home():
            return render_template("index.html", title="Home")

        @app.route("/about")
        def about():
            return render_template("about.html", title="About")

        @app.route("/contact", methods=["GET", "POST"])
        def contact():
            if request.method == "POST":
                name = request.form.get("name")
                email = request.form.get("email")
                message = request.form.get("message")

                db = get_db()
                cursor = db.cursor()
                cursor.execute("INSERT INTO messages (name, email, message) VALUES (?, ?, ?)",
                               (name, email, message))
                db.commit()

                return render_template("contact.html", title="Contact", success=True, name=name)
            return render_template("contact.html", title="Contact", success=False)

        @app.route("/admin")
        def admin():
            db = get_db()
            cursor = db.cursor()
            cursor.execute("SELECT * FROM message ORDER BY id DESC")
            messages = cursor.fetchall()
            return render_template("admin.html", title="Admin Dashboard", messages=messages)

        if __name__ == "__main__":
            init_db()
            app.run(debug=True)

├── requirements.txt
│   └──
        Flask

├── templates/
│   ├── base.html
│   │   └──
                <!DOCTYPE html>
                <html lang="en">
                <head>
                    <meta charset="UTF-8">
                    <meta name="viewport" content="width=device-width, initial-scale=1.0">
                    <title>{{ title if title else "Flask App" }}</title>
                    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
                </head>
                <body>
                    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
                        <div class="container">
                            <a class="navbar-brand" href="/">Flask App</a>
                            <div>
                                <a class="nav-link d-inline text-white" href="/">Home</a>
                                <a class="nav-link d-inline text-white" href="/about">About</a>
                                <a class="nav-link d-inline text-white" href="/contact">Contact</a>
                                <a class="nav-link d-inline text-warning" href="/admin">Admin</a>
                            </div>
                        </div>
                    </nav>
                    <div class="container mt-4">
                        {% block content %}{% endblock %}
                    </div>
                </body>
                </html>
│
│   ├── index.html
│   │   └──
                {% extends "base.html" %}
                {% block content %}
                <h1>Welcome to the Flask App</h1>
                <p class="lead">A Flask application with a database and admin dashboard.</p>
                {% endblock %}
│
│   ├── about.html
│   │   └──
                {% extends "base.html" %}
                {% block content %}
                <h1>About</h1>
                <p>This app demonstrates Flask with multiple pages, a contact form, SQLite database, and an admin dashboard.</p>
                {% endblock %}
│
│   ├── contact.html
│   │   └──
                {% extends "base.html" %}
                {% block content %}
                <h1>Contact Us</h1>
                {% if success %}
                    <div class="alert alert-success">Thank you, {{ name }}! Your message has been saved.</div>
                {% else %}
                    <form method="POST">
                        <div class="mb-3">
                            <label class="form-label">Name</label>
                            <input type="text" class="form-control" name="name" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Email</label>
                            <input type="email" class="form-control" name="email" required>
                        </div>
                        <div class="mb-3">
                            <label class="form-label">Message</label>
                            <textarea class="form-control" name="message" rows="4" required></textarea>
                        </div>
                        <button type="submit" class="btn btn-primary">Send</button>
                    </form>
                {% endif %}
                {% endblock %}
│
│   └── admin.html
│       └──
                {% extends "base.html" %}
                {% block content %}
                <h1>Admin Dashboard</h1>
                {% if messages %}
                    <table class="table table-bordered">
                        <thead>
                            <tr>
                                <th>ID</th>
                                <th>Name</th>
                                <th>Email</th>
                                <th>Message</th>
                            </tr>
                        </thead>
                        <tbody>
                            {% for msg in messages %}
                                <tr>
                                    <td>{{ msg.id }}</td>
                                    <td>{{ msg.name }}</td>
                                    <td>{{ msg.email }}</td>
                                    <td>{{ msg.message }}</td>
                                </tr>
                            {% endfor %}
                        </tbody>
                    </table>
                {% else %}
                    <p>No messages found.</p>
                {% endif %}
                {% endblock %}

└── README.md
    └──
        # Flask Web App with Admin Dashboard

        This Flask app includes:
        - Multiple pages (Home, About, Contact, Admin)
        - Bootstrap styling
        - Contact form with SQLite database storage
        - Admin dashboard to view saved messages

        ## Installation
        ```bash
        pip install -r requirements.txt
        python app.py
        ```

        Then visit:
        ```
        http://127.0.0.1:50
        ```
