# Universal-Interaction-Platform
The instructions include necessary commands, code, and explanations for setting up the application step by step.
## Backend Setup (FastAPI)
The backend handles user authentication, messaging, and a simple newsfeed.
-----------------------------------------
### Code:app.py
```python
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
```
### Dependencies: requirements.txt
fastapi
uvicorn
pydantic
sqlite3

### Commands to Set Up and Run the Backend
1.Install dependencies

pip install -r requirements.txt

2.Run the server

uvicorn app:app --reload

The backend will now be available at http://localhost:8000

### Frontend Setup (React.js)
The frontend allows users to interact with the backend through a simple interface.
### Code: App.js
```javascript
import React, { useState } from "react";

const App = () => {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [token, setToken] = useState(false);
  const [message, setMessage] = useState("");
  const [recipient, setRecipient] = useState("");
  const [newsfeed, setNewsfeed] = useState([]);

  const signup = async () => {
    try {
      const response = await fetch("http://localhost:8000/signup/", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ username, password }),
      });
      const data = await response.json();
      alert(data.message);
    } catch (error) {
      console.error(error);
    }
  };

  const login = async () => {
    try {
      const response = await fetch("http://localhost:8000/login/", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ username, password }),
      });
      const data = await response.json();
      if (response.ok) {
        alert(data.message);
        setToken(true);
      } else {
        alert(data.detail);
      }
    } catch (error) {
      console.error(error);
    }
  };

  const sendMessage = async () => {
    try {
      const response = await fetch("http://localhost:8000/send-message/", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ sender: username, recipient, content: message }),
      });
      const data = await response.json();
      alert(data.message);
    } catch (error) {
      console.error(error);
    }
  };

  const fetchNewsfeed = async () => {
    try {
      const response = await fetch("http://localhost:8000/newsfeed/");
      const data = await response.json();
      setNewsfeed(data);
    } catch (error) {
      console.error(error);
    }
  };

  return (
    <div>
      <h1>Universal Interaction Platform</h1>
      <div>
        <input placeholder="Username" value={username} onChange={(e) => setUsername(e.target.value)} />
        <input placeholder="Password" value={password} onChange={(e) => setPassword(e.target.value)} type="password" />
        <button onClick={signup}>Sign Up</button>
        <button onClick={login}>Log In</button>
      </div>

      {token && (
        <div>
          <h2>Messages</h2>
          <input placeholder="Recipient" value={recipient} onChange={(e) => setRecipient(e.target.value)} />
          <textarea placeholder="Message" value={message} onChange={(e) => setMessage(e.target.value)} />
          <button onClick={sendMessage}>Send</button>

          <h2>Newsfeed</h2>
          <button onClick={fetchNewsfeed}>Refresh Newsfeed</button>
          {newsfeed.map((post, index) => (
            <div key={index}>
              <strong>{post.author}:</strong> {post.content}
            </div>
          ))}
        </div>
      )}
    </div>
  );
};

export default App;
```




