---

# Lesson: Ubuntu Server Setup, Deployment, and Reverse Proxy

## 1. Connecting to the Server
First, secure your private key and connect via SSH. Replace the `.pem` filename and IP address with your own.

```bash
# Set correct permissions for the private key
chmod 400 emmanuel-emmanuel-1774290299806.pem

# SSH into the server
ssh -i emmanuel-emmanuel-1774290299806.pem emmanuel@34.13.9.251
```

## 2. System Update & Initial Tools
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nano -y
```

## 3. Setting up Node.js Environment
```bash
sudo apt install nodejs npm -y

# Verify installation
node -v
npm -v
```

### Create a Test Node.js Server
`nano server.js`
```javascript
const http = require('http');

http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from your Ubuntu server!');
}).listen(3000, '0.0.0.0', () => {
  console.log('Server running on port 3000');
});
```

## 4. Setting up Python & Flask
```bash
sudo apt install python3-venv -y
python3 -m venv venv
source venv/bin/activate
pip install flask
```

### Create a Test Flask App
`nano app.py`
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return "Hello from your Ubuntu server!"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=3000)
```

## 5. Deploying from Git
```bash
sudo mkdir -p /var/www/
sudo chown -R $USER:$USER /var/www
cd /var/www/
git clone https://github.com/emmanuel-omegaalexis/node_placeholder.git
cd node_placeholder
npm install
```

## 6. Process Management with PM2
PM2 allows you to keep your apps running in the background.

### For Node.js Apps:
```bash
sudo npm install -g pm2
pm2 start index.js --name my-node-app
```

### For Python Apps:
To run a Python app, you must tell PM2 which interpreter to use:
```bash
pm2 start app.py --interpreter python3 --name my-python-app
```

### Useful PM2 Commands:
```bash
pm2 list          # See all running apps
pm2 logs          # View real-time output/errors
pm2 stop all      # Stop all apps
pm2 save          # Save list for auto-restart on reboot
pm2 startup       # Generates script to start PM2 on boot
```

## 7. Configuring Nginx as a Reverse Proxy
```bash
sudo apt install nginx -y
sudo nano /etc/nginx/sites-available/default
```

**Clear the file and paste:**
```nginx
server {
    listen 80;
    server_name YOUR_DOMAIN_OR_SERVER_IP;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
`sudo nginx -t` && `sudo systemctl restart nginx`

## 8. Security (Firewall)
```bash
sudo apt install ufw -y
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

## 9. Local Domain Simulation (Hosts file)
**On your Local Machine (Mac/Linux):**
`sudo nano /etc/hosts` → Add `34.13.9.251 myapp.local`

**On your Local Machine (Windows):**
Open Notepad as Admin → Open `C:\Windows\System32\drivers\etc\hosts` → Add `34.13.9.251 myapp.local`

## 10. SSL Setup (Certbot)
```bash
sudo apt install certbot python3-certbot-nginx -y

# Change 'server_name' in Nginx config to your real domain first!
sudo certbot --nginx -d emmanuel.omegaalexis.site
```
