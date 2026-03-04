# 🚀 Gradio Interface Deployment Guide

## Running Gradio on GitHub Pages - Complete Guide

**Important Note:** GitHub Pages serves static HTML/CSS/JS files only. Gradio apps require a Python backend server, so you have several deployment options:

---

## 📋 Deployment Options

### Option 1: Hugging Face Spaces (Recommended - FREE & Easy)

**Best for:** Quick deployment, free hosting, automatic HTTPS

#### Steps:

1. **Create Hugging Face Account**
   - Go to https://huggingface.co/join
   - Create free account

2. **Create New Space**
   ```bash
   # Go to: https://huggingface.co/new-space
   # Choose:
   # - Space name: qa-bot-langchain
   # - License: MIT
   # - SDK: Gradio
   # - Hardware: CPU basic (free)
   ```

3. **Upload Your Files**
   ```bash
   # Clone your space
   git clone https://huggingface.co/spaces/YOUR_USERNAME/qa-bot-langchain
   cd qa-bot-langchain
   
   # Add your files
   # app.py (your Gradio app)
   # requirements.txt
   # README.md
   
   # Push to Hugging Face
   git add .
   git commit -m "Initial commit"
   git push
   ```

4. **Create `app.py`**
   ```python
   import gradio as gr
   
   def process_image(image):
       # Your image processing logic
       return image
   
   demo = gr.Interface(
       fn=process_image,
       inputs=gr.Image(type="pil"),
       outputs=gr.Image(type="pil"),
       title="Image Processor"
   )
   
   demo.launch()
   ```

5. **Create `requirements.txt`**
   ```txt
   gradio==4.0.0
   pillow
   numpy
   # Add other dependencies
   ```

6. **Embed in Your GitHub Pages**
   ```html
   <!-- In qa-langchain.html -->
   <iframe 
       class="gradio-iframe" 
       src="https://YOUR_USERNAME-qa-bot-langchain.hf.space"
       title="QA Bot Interface"
   ></iframe>
   ```

---

### Option 2: GitHub Actions + External Hosting

**Best for:** Custom deployment, CI/CD integration

#### A. Deploy to Render.com (Free Tier)

1. **Create `render.yaml`**
   ```yaml
   services:
     - type: web
       name: gradio-app
       env: python
       buildCommand: pip install -r requirements.txt
       startCommand: python app.py
       envVars:
         - key: PYTHON_VERSION
           value: 3.10.0
   ```

2. **GitHub Actions Workflow** (`.github/workflows/deploy.yml`)
   ```yaml
   name: Deploy Gradio App
   
   on:
     push:
       branches: [main]
   
   jobs:
     deploy:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         
         - name: Deploy to Render
           env:
             RENDER_API_KEY: ${{ secrets.RENDER_API_KEY }}
           run: |
             curl -X POST \
               https://api.render.com/v1/services/$SERVICE_ID/deploys \
               -H "Authorization: Bearer $RENDER_API_KEY"
   ```

#### B. Deploy to Railway.app

1. **Install Railway CLI**
   ```bash
   npm i -g @railway/cli
   ```

2. **Deploy**
   ```bash
   railway login
   railway init
   railway up
   ```

3. **GitHub Actions** (`.github/workflows/railway.yml`)
   ```yaml
   name: Deploy to Railway
   
   on:
     push:
       branches: [main]
   
   jobs:
     deploy:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         
         - name: Install Railway
           run: npm i -g @railway/cli
         
         - name: Deploy
           env:
             RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
           run: railway up
   ```

---

### Option 3: Self-Hosted with GitHub Actions

**Best for:** Full control, custom infrastructure

#### Setup:

1. **Create `Dockerfile`**
   ```dockerfile
   FROM python:3.10-slim
   
   WORKDIR /app
   
   COPY requirements.txt .
   RUN pip install --no-cache-dir -r requirements.txt
   
   COPY . .
   
   EXPOSE 7860
   
   CMD ["python", "app.py"]
   ```

2. **GitHub Actions** (`.github/workflows/docker-deploy.yml`)
   ```yaml
   name: Build and Deploy Docker
   
   on:
     push:
       branches: [main]
   
   jobs:
     build-and-deploy:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         
         - name: Build Docker Image
           run: docker build -t gradio-app .
         
         - name: Push to Registry
           env:
             DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
             DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
           run: |
             echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
             docker tag gradio-app $DOCKER_USERNAME/gradio-app:latest
             docker push $DOCKER_USERNAME/gradio-app:latest
         
         - name: Deploy to Server
           uses: appleboy/ssh-action@master
           with:
             host: ${{ secrets.SERVER_HOST }}
             username: ${{ secrets.SERVER_USER }}
             key: ${{ secrets.SSH_PRIVATE_KEY }}
             script: |
               docker pull ${{ secrets.DOCKER_USERNAME }}/gradio-app:latest
               docker stop gradio-app || true
               docker rm gradio-app || true
               docker run -d -p 7860:7860 --name gradio-app \
                 ${{ secrets.DOCKER_USERNAME }}/gradio-app:latest
   ```

---

## 🎯 Complete Example: Image Upload Gradio App

### File Structure:
```
your-repo/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── app.py
├── requirements.txt
├── README.md
└── qa-langchain.html (your GitHub Pages file)
```

### `app.py` - Simple Image Upload Example:
```python
import gradio as gr
from PIL import Image
import numpy as np

def process_image(image):
    """Process uploaded image"""
    if image is None:
        return None, "No image uploaded"
    
    # Convert to numpy array
    img_array = np.array(image)
    
    # Example: Convert to grayscale
    gray = np.mean(img_array, axis=2).astype(np.uint8)
    gray_img = Image.fromarray(gray)
    
    info = f"Image size: {image.size}\nMode: {image.mode}"
    
    return gray_img, info

# Create interface
demo = gr.Interface(
    fn=process_image,
    inputs=gr.Image(type="pil", label="Upload Image"),
    outputs=[
        gr.Image(type="pil", label="Processed Image"),
        gr.Textbox(label="Image Info")
    ],
    title="Image Processor",
    description="Upload an image to process it"
)

if __name__ == "__main__":
    demo.launch(server_name="0.0.0.0", server_port=7860)
```

### `requirements.txt`:
```txt
gradio==4.0.0
pillow==10.0.0
numpy==1.24.0
```

---

## 🔧 Local Development

### Run Locally:
```bash
# Install dependencies
pip install -r requirements.txt

# Run app
python app.py

# Access at: http://localhost:7860
```

### Test with ngrok (Public URL):
```bash
# Install ngrok
brew install ngrok  # macOS
# or download from https://ngrok.com

# Run your app
python app.py

# In another terminal, create tunnel
ngrok http 7860

# Use the ngrok URL in your iframe
```

---

## 🌐 Embed in GitHub Pages

### Update `qa-langchain.html`:

```html
<!-- Replace placeholder with your deployed URL -->
<iframe 
    class="gradio-iframe" 
    src="https://YOUR_USERNAME-qa-bot.hf.space"
    title="QA Bot Interface"
    allow="camera; microphone"
></iframe>
```

### Deployment URLs by Platform:
- **Hugging Face**: `https://YOUR_USERNAME-SPACE_NAME.hf.space`
- **Render**: `https://your-app-name.onrender.com`
- **Railway**: `https://your-app.railway.app`
- **Custom**: `https://your-domain.com`

---

## 🔐 Environment Variables

### Set API Keys (if needed):

#### Hugging Face Spaces:
```bash
# In Space Settings > Variables
OPENAI_API_KEY=your-key-here
```

#### GitHub Actions:
```bash
# In Repo Settings > Secrets
OPENAI_API_KEY
RENDER_API_KEY
RAILWAY_TOKEN
```

#### Local Development:
```bash
export OPENAI_API_KEY='your-key-here'
python app.py
```

---

## 📊 Monitoring & Logs

### Hugging Face Spaces:
- View logs in Space settings
- Monitor usage in dashboard

### GitHub Actions:
- Check workflow runs in Actions tab
- View deployment logs

### Self-Hosted:
```bash
# Docker logs
docker logs -f gradio-app

# System logs
journalctl -u gradio-app -f
```

---

## 🐛 Troubleshooting

### Common Issues:

1. **CORS Errors**
   ```python
   # In app.py
   demo.launch(
       server_name="0.0.0.0",
       allowed_paths=["*"],
       root_path="/gradio"
   )
   ```

2. **Port Already in Use**
   ```bash
   # Kill process on port 7860
   lsof -ti:7860 | xargs kill -9
   ```

3. **Dependencies Not Installing**
   ```txt
   # Pin versions in requirements.txt
   gradio==4.0.0
   pillow==10.0.0
   ```

---

## 📚 Additional Resources

- [Gradio Documentation](https://gradio.app/docs)
- [Hugging Face Spaces Guide](https://huggingface.co/docs/hub/spaces)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Documentation](https://docs.docker.com)

---

## ✅ Quick Start Checklist

- [ ] Choose deployment platform (Hugging Face recommended)
- [ ] Create `app.py` with Gradio interface
- [ ] Create `requirements.txt` with dependencies
- [ ] Test locally with `python app.py`
- [ ] Deploy to chosen platform
- [ ] Get deployment URL
- [ ] Update `qa-langchain.html` iframe src
- [ ] Push to GitHub Pages
- [ ] Test embedded interface

---

**Need Help?** Open an issue in your repository or check the platform-specific documentation.