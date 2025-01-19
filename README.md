# Universal-Interaction-Platform
The instructions include necessary commands, code, and explanations for setting up the application step by step.
## Backend Setup (FastAPI)
The backend handles user authentication, messaging, and a simple newsfeed.
-----------------------------------------
### Code:app.py
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import List
import sqlite3

app = FastAPI()

# Database setup
DATABASE = "database.db"

def init_db():
    """Initialize the database with tables for users, messages, and posts."""
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()
    cursor.execute('''CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, username TEXT, password TEXT)''')
    cursor.execute('''CREATE TABLE IF NOT EXISTS messages (id INTEGER PRIMARY KEY, sender TEXT, recipient TEXT, content TEXT)''')
    cursor.execute('''CREATE TABLE IF NOT EXISTS posts (id INTEGER PRIMARY KEY, author TEXT, content TEXT)''')
    conn.commit()
    conn.close()

init_db()

# Models
class User(BaseModel):
    username: str
    password: str

class Message(BaseModel):
    sender: str
    recipient: str
    content: str

class Post(BaseModel):
    author: str
    content: str

# Routes
@app.post("/signup/")
def signup(user: User):
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE username = ?", (user.username,))
    if cursor.fetchone():
        raise HTTPException(status_code=400, detail="Username already exists.")
    cursor.execute("INSERT INTO users (username, password) VALUES (?, ?)", (user.username, user.password))
    conn.commit()
    conn.close()
    return {"message": "User created successfully."}

@app.post("/login/")
def login(user: User):
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE username = ? AND password = ?", (user.username, user.password))
    if not cursor.fetchone():
        raise HTTPException(status_code=400, detail="Invalid credentials.")
    conn.close()
    return {"message": "Login successful."}

@app.post("/send-message/")
def send_message(message: Message):
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()
    cursor.execute("INSERT INTO messages (sender, recipient, content) VALUES (?, ?, ?)",
                   (message.sender, message.recipient, message.content))
    conn.commit()
    conn.close()
    return {"message": "Message sent successfully."}

@app.get("/messages/{username}/", response_model=List[Message])
def get_messages(username: str):
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()
    cursor.execute("SELECT sender, recipient, content FROM messages WHERE recipient = ?", (username,))
    messages = cursor.fetchall()
    conn.close()
    return [{"sender": m[0], "recipient": m[1], "content": m[2]} for m in messages]

@app.post("/create-post/")
def create_post(post: Post):
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()
    cursor.execute("INSERT INTO posts (author, content) VALUES (?, ?)", (post.author, post.content))
    conn.commit()
    conn.close()
    return {"message": "Post created successfully."}

@app.get("/newsfeed/", response_model=List[Post])
def get_newsfeed():
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()
    cursor.execute("SELECT author, content FROM posts")
    posts = cursor.fetchall()
    conn.close()
    return [{"author": p[0], "content": p[1]} for p in posts]


