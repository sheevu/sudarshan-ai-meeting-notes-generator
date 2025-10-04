# 🚀 Deployment Guide - Sudarshan AI Labs Meeting Notes App

## 📋 Quick Overview

**3 Deployment Options:**
1. **Vercel + Anthropic API** (Easiest, Recommended)
2. **Netlify + Serverless Functions**
3. **Self-Hosted (VPS/AWS/Digital Ocean)**

---

## ✅ Option 1: Vercel Deployment (RECOMMENDED)

### **Why Vercel?**
- ✅ Free tier available
- ✅ Automatic HTTPS
- ✅ Easy serverless functions
- ✅ Fast deployment
- ✅ Great for React apps

### **Step-by-Step:**

#### 1️⃣ **Get an AI API Key**

Choose one:

**Anthropic Claude** (Best for summarization)
```
1. Visit: https://console.anthropic.com/
2. Sign up / Login
3. Go to "API Keys"
4. Create new key
5. Copy the key (starts with sk-ant-...)
6. Cost: ~$3 per 1 million tokens (very affordable)
```

**OpenAI GPT** (Alternative)
```
1. Visit: https://platform.openai.com/
2. Sign up / Login
3. Go to "API Keys"
4. Create new key
5. Cost: ~$5 per 1 million tokens
```

#### 2️⃣ **Setup Project Structure**

Create this folder structure:
```
sudarshan-meeting-notes/
├── src/
│   └── App.jsx          (React frontend - use code from artifact)
├── api/
│   └── summarize.js     (Backend API)
├── public/
│   └── index.html
├── package.json
├── vite.config.js
├── .env.local          (API keys - DON'T commit!)
└── vercel.json
```

#### 3️⃣ **Create Files**

**api/summarize.js**
```javascript
import Anthropic from '@anthropic-ai/sdk';

export default async function handler(req, res) {
  // Enable CORS
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
  
  if (req.method === 'OPTIONS') {
    return res.status(200).end();
  }

  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { notes, language } = req.body;

    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });

    const prompt = language === 'hi-IN' 
      ? `कृपया निम्नलिखित मीटिंग नोट्स को एक संरचित सारांश में बदलें:\n\n${notes}`
      : `Please convert the following meeting notes into a well-structured summary with clear headings and bullet points:\n\n${notes}`;

    const message = await anthropic.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 2000,
      messages: [{ role: 'user', content: prompt }]
    });

    res.status(200).json({
      summary: message.content[0].text
    });

  } catch (error) {
    console.error('Error:', error);
    res.status(500).json({ 
      error: 'Failed to generate summary',
      details: error.message 
    });
  }
}
```

**package.json**
```json
{
  "name": "sudarshan-meeting-notes",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "@anthropic-ai/sdk": "^0.20.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.0.0",
    "vite": "^4.3.9"
  }
}
```

**vite.config.js**
```javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': 'http://localhost:3000'
    }
  }
});
```

**vercel.json**
```json
{
  "rewrites": [
    { "source": "/api/(.*)", "destination": "/api/$1" }
  ]
}
```

**.env.local** (NEVER commit this!)
```
ANTHROPIC_API_KEY=sk-ant-your-key-here
```

#### 4️⃣ **Deploy to Vercel**

**Method A: Using Vercel CLI**
```bash
# Install Vercel CLI
npm install -g vercel

# Login to Vercel
vercel login

# Deploy
vercel

# Follow prompts:
# - Link to existing project? N
# - What's your project name? sudarshan-meeting-notes
# - In which directory? ./
# - Override settings? N

# Add environment variable
vercel env add ANTHROPIC_API_KEY

# Paste your API key when prompted

# Deploy to production
vercel --prod
```

**Method B: Using Vercel Dashboard**
```
1. Go to https://vercel.com
2. Click "Add New Project"
3. Import your GitHub repository
4. Add environment variable:
   - Name: ANTHROPIC_API_KEY
   - Value: your-api-key
5. Click "Deploy"
```

---

## ✅ Option 2: Netlify Deployment

### **Step-by-Step:**

#### 1️⃣ **Create Netlify Function**

**netlify/functions/summarize.js**
```javascript
import Anthropic from '@anthropic-ai/sdk';

exports.handler = async (event, context) => {
  if (event.httpMethod !== 'POST') {
    return { statusCode: 405, body: 'Method Not Allowed' };
  }

  try {
    const { notes, language } = JSON.parse(event.body);

    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });

    const prompt = language === 'hi-IN' 
      ? `कृपया निम्नलिखित मीटिंग नोट्स को एक संरचित सारांश में बदलें:\n\n${notes}`
      : `Please convert the following meeting notes into a structured summary:\n\n${notes}`;

    const message = await anthropic.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 2000,
      messages: [{ role: 'user', content: prompt }]
    });

    return {
      statusCode: 200,
      body: JSON.stringify({
        summary: message.content[0].text
      })
    };
  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: 'Failed to generate summary' })
    };
  }
};
```

#### 2️⃣ **Deploy**
```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login
netlify login

# Initialize
netlify init

# Deploy
netlify deploy --prod

# Add environment variable in Netlify dashboard:
# Site settings > Environment variables > Add variable
# ANTHROPIC_API_KEY = your-key
```

---

## ✅ Option 3: Self-Hosted (VPS)

### **For AWS EC2 / Digital Ocean / Any VPS:**

#### 1️⃣ **Create Full Backend**

**server.js**
```javascript
import express from 'express';
import Anthropic from '@anthropic-ai/sdk';
import cors from 'cors';
import 'dotenv/config';

const app = express();
app.use(cors());
app.use(express.json());
app.use(express.static('dist')); // Serve built React app

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

app.post('/api/summarize', async (req, res) => {
  try {
    const { notes, language } = req.body;

    const prompt = language === 'hi-IN' 
      ? `कृपया निम्नलिखित मीटिंग नोट्स को एक संरचित सारांश में बदलें:\n\n${notes}`
      : `Please convert the following meeting notes into a structured summary:\n\n${notes}`;

    const message = await anthropic.messages.create({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 2000,
      messages: [{ role: 'user', content: prompt }]
    });

    res.json({ summary: message.content[0].text });
  } catch (error) {
    console.error('Error:', error);
    res.status(500).json({ error: 'Failed to generate summary' });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`🚀 Server running on port ${PORT}`);
});
```

#### 2️⃣ **Deploy on VPS**
```bash
# SSH into your server
ssh user@your-server-ip

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Clone your repository
git clone your-repo-url
cd sudarshan-meeting-notes

# Install dependencies
npm install

# Build frontend
npm run build

# Create .env file
nano .env
# Add: ANTHROPIC_API_KEY=your-key

# Install PM2 (process manager)
sudo npm install -g pm2

# Start server
pm2 start server.js
pm2 save
pm2 startup

# Setup Nginx as reverse proxy
sudo apt install nginx
sudo nano /etc/nginx/sites-available/default

# Add this configuration:
# server {
#   listen 80;
#   server_name your-domain.com;
#   location / {
#     proxy_pass http://localhost:3000;
#   }
# }

sudo systemctl restart nginx
```

---

## 💰 Cost Comparison

| Platform | Hosting Cost | AI API Cost | Total/Month |
|----------|--------------|-------------|-------------|
| **Vercel** | Free (hobby) | ~$3-10 | $3-10 |
| **Netlify** | Free (starter) | ~$3-10 | $3-10 |
| **AWS EC2** | $5-10 | ~$3-10 | $8-20 |
| **Digital Ocean** | $5 | ~$3-10 | $8-15 |

**AI API Usage:**
-