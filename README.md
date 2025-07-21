# MERN Stack Capstone Project

This assignment focuses on designing, developing, and deploying a comprehensive full-stack MERN application that showcases all the skills you've learned throughout the course.

## Assignment Overview

You will:
1. Plan and design a full-stack MERN application
2. Develop a robust backend with MongoDB, Express.js, and Node.js
3. Create an interactive frontend with React.js
4. Implement testing across the entire application
5. Deploy the application to production

## Getting Started

1. Accept the GitHub Classroom assignment
2. Clone the repository to your local machine
3. Follow the instructions in the `Week8-Assignment.md` file
4. Plan, develop, and deploy your capstone project

## Files Included

- `Week8-Assignment.md`: Detailed assignment instructions

## Requirements

- Node.js (v18 or higher)
- MongoDB (local installation or Atlas account)
- npm or yarn
- Git and GitHub account
- Accounts on deployment platforms (Render/Vercel/Netlify/etc.)

## Project Ideas

The `Week8-Assignment.md` file includes several project ideas, but you're encouraged to develop your own idea that demonstrates your skills and interests.

## Submission

Your project will be automatically submitted when you push to your GitHub Classroom repository. Make sure to:

1. Commit and push your code regularly
2. Include comprehensive documentation
3. Deploy your application and add the live URL to your README.md
4. Create a video demonstration and include the link in your README.md

## Resources

- [MongoDB Documentation](https://docs.mongodb.com/)
- [Express.js Documentation](https://expressjs.com/)
- [React Documentation](https://react.dev/)
- [Node.js Documentation](https://nodejs.org/en/docs/)
- [GitHub Classroom Guide](https://docs.github.com/en/education/manage-coursework-with-github-classroom)


# MindSpace: MERN Mental Health Journal

Welcome to **MindSpace**, a heartfelt full-stack MERN application crafted by **Tiffany Nyambura Wainaina**. This project was created while navigating illness — a reminder that sometimes healing and learning happen at the same time. MindSpace is a personal and public mental health journaling app that helps users express emotions, reflect, and share real experiences with others.

---

## 🧠 Why MindSpace?

Mental health often goes unspoken — but what if writing down your thoughts could be the first step to healing? MindSpace was designed as a safe space to:

* Log private thoughts and emotions
* Share public journal entries to inspire others
* Track mood patterns over time
* Foster connection through empathy

It combines **psychology-inspired journaling** with **modern web development**, showcasing not just technical ability, but emotional depth.

---

## 💻 Tech Stack

**Frontend:**

* React (Vite)
* Tailwind CSS
* Axios
* Socket.IO Client

**Backend:**

* Express.js
* MongoDB with Mongoose
* Socket.IO
* JWT Authentication

**Tools:**

* dotenv
* bcryptjs
* nodemon (dev)

---

## ✨ Key Features

* 📓 **Journal Entries**: Create, view, and share mental health entries
* 😌 **Mood Selector**: Tag your thoughts with an emoji mood (e.g., 😊, 😢, 😔)
* 🔐 **Authentication**: Register/login securely
* 🔁 **Real-Time Updates**: New public entries appear instantly via WebSockets
* 🌐 **Public Feed**: View anonymous entries shared by others

---

## ⚙️ Getting Started

1. **Clone the Repository**

```bash
git clone https://github.com/yourusername/mindspace.git
cd mindspace
```

2. **Backend Setup**

```bash
cd server
npm install
```

Create a `.env` file and add:

```env
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret
```

Start the backend server:

```bash
npm run dev
```

3. **Frontend Setup**

```bash
cd ../client
npm install
npm run dev
```

---

## 🧪 Testing

* Simple error messages and manual testing implemented.
* Project was developed under real conditions — including imperfections.

---

## 🙋‍♀️ About Me

I'm **Tiffany Nyambura Wainaina**, a psychology enthusiast and developer passionate about merging technology and emotional wellness. MindSpace was born from my own healing journey — it’s more than code, it’s a message.

---

## 📌 Future Improvements

* Add user profile and entry history
* Analyze mood trends using charts
* Mobile app version using React Native
* AI-powered journaling suggestions

---


// === Backend: server/index.js ===
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const http = require('http');
const { Server } = require('socket.io');
require('dotenv').config();

const authRoutes = require('./routes/auth');
const journalRoutes = require('./routes/journals');

const app = express();
const server = http.createServer(app);
const io = new Server(server, { cors: { origin: '*' } });

mongoose.connect(process.env.MONGO_URI)
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.error(err));

app.use(cors());
app.use(express.json());

app.use('/api/auth', authRoutes);
app.use('/api/journals', journalRoutes(io));

server.listen(5000, () => console.log('Server running on port 5000'));


// === Backend: models/User.js ===
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const UserSchema = new mongoose.Schema({
  username: { type: String, required: true, unique: true },
  password: { type: String, required: true }
});

UserSchema.pre('save', async function () {
  if (!this.isModified('password')) return;
  this.password = await bcrypt.hash(this.password, 10);
});

module.exports = mongoose.model('User', UserSchema);


// === Backend: models/Journal.js ===
const mongoose = require('mongoose');

const JournalSchema = new mongoose.Schema({
  userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  content: { type: String, required: true },
  emotion: { type: String },
  isPublic: { type: Boolean, default: false },
  createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Journal', JournalSchema);


// === Backend: routes/auth.js ===
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');
const User = require('../models/User');

const router = express.Router();

router.post('/register', async (req, res) => {
  const user = new User(req.body);
  await user.save();
  res.status(201).send('User registered');
});

router.post('/login', async (req, res) => {
  const user = await User.findOne({ username: req.body.username });
  if (!user || !(await bcrypt.compare(req.body.password, user.password))) {
    return res.status(401).send('Invalid credentials');
  }
  const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET);
  res.json({ token });
});

module.exports = router;


// === Backend: routes/journals.js ===
const express = require('express');
const Journal = require('../models/Journal');
const jwt = require('jsonwebtoken');

module.exports = function (io) {
  const router = express.Router();

  const auth = (req, res, next) => {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) return res.status(403).send('No token');
    try {
      req.user = jwt.verify(token, process.env.JWT_SECRET);
      next();
    } catch {
      res.status(403).send('Invalid token');
    }
  };

  router.get('/', async (req, res) => {
    const journals = await Journal.find({ isPublic: true }).sort({ createdAt: -1 });
    res.json(journals);
  });

  router.post('/', auth, async (req, res) => {
    const journal = new Journal({ ...req.body, userId: req.user.id });
    await journal.save();
    io.emit('new-journal', journal);
    res.status(201).json(journal);
  });

  return router;
};


// === Frontend: src/App.jsx ===
import { useEffect, useState } from 'react';
import io from 'socket.io-client';
import axios from 'axios';

const socket = io('http://localhost:5000');

function App() {
  const [journals, setJournals] = useState([]);
  const [form, setForm] = useState({ content: '', emotion: '', isPublic: false });

  useEffect(() => {
    axios.get('/api/journals').then(res => setJournals(res.data));
    socket.on('new-journal', journal => setJournals(prev => [journal, ...prev]));
  }, []);

  const submit = async () => {
    const token = localStorage.getItem('token');
    await axios.post('/api/journals', form, {
      headers: { Authorization: `Bearer ${token}` }
    });
    setForm({ content: '', emotion: '', isPublic: false });
  };

  return (
    <div className="p-6">
      <textarea value={form.content} onChange={e => setForm({ ...form, content: e.target.value })} />
      <input value={form.emotion} onChange={e => setForm({ ...form, emotion: e.target.value })} />
      <label>
        <input type="checkbox" checked={form.isPublic} onChange={e => setForm({ ...form, isPublic: e.target.checked })} />
        Share Publicly
      </label>
      <button onClick={submit}>Post</button>

      <div className="mt-4">
        {journals.map((j, i) => (
          <div key={i} className="border p-2 my-2">
            <p>{j.content}</p>
            <small>{j.emotion}</small>
          </div>
        ))}
      </div>
    </div>
  );
}

export default App;


## 🧡 Final Note

This project is a tribute to anyone quietly battling anxiety, stress, or sadness. I hope MindSpace becomes more than just an app — maybe a moment of comfort in someone’s day.

> "It's okay to not be okay — just don't stop showing up for yourself."

---

Thank you for reading 🌿
