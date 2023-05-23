# Python (Flask) example
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_bcrypt import Bcrypt
from flask_login import LoginManager

app = Flask(__name__)
db = SQLAlchemy(app)
bcrypt = Bcrypt(app)
login_manager = LoginManager(app)

# User model
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(30), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password = db.Column(db.String(60), nullable=False)

# Register route
@app.route("/register", methods=['POST'])
def register():
    username = request.json.get('username')
    email = request.json.get('email')
    password = bcrypt.generate_password_hash(request.json.get('password')).decode('utf-8')
    
    user = User(username=username, email=email, password=password)
    db.session.add(user)
    db.session.commit()

    return jsonify(message="User created."), 201

# Login route
@app.route("/login", methods=['POST'])
def login():
    user = User.query.filter_by(email=request.json.get('email')).first()
    
    if user and bcrypt.check_password_hash(user.password, request.json.get('password')):
        login_user(user)
        return jsonify(message="Logged in successfully."), 200

    return jsonify(message="Invalid credentials."), 401
