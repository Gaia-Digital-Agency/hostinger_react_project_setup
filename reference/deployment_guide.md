# Project Setup and Server Deployment Guide

This document outlines the complete process for setting up the React project locally and deploying it to a Debian server with live, staging, and development environments.

## 1. Local Project Setup

This section covers the creation of the React project on your local machine.

### 1.1. Create the Project

A new React project was created using Vite with the TypeScript template.

```bash
npm create vite@latest site02.com -- --template react-ts
```

### 1.2. Add Tailwind CSS

Tailwind CSS and its peer dependencies were installed.

```bash
# Navigate into the new project directory
cd site02.com

# Install dependencies
npm install -D tailwindcss@^3 postcss@^8 autoprefixer@^10
```

The project was initialized for Tailwind, creating `tailwind.config.js` and `postcss.config.js`.

```bash
npx tailwindcss init -p
```

The `tailwind.config.js` was updated to scan project files:

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

The main CSS file (`src/index.css`) was updated with Tailwind's directives:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
/* ... existing CSS ... */
```

### 1.3. Add SCSS Support

Sass was installed to enable SCSS.

```bash
npm install -D sass
```

The `src/App.css` file was renamed to `src/App.scss` and the import in `src/App.tsx` was updated accordingly.

### 1.4. Initialize Git Repository

A Git repository was initialized with three branches for the different environments.

```bash
git init
git add .
git commit -m "Initial commit with React, Vite, Tailwind, and SCSS setup"
git branch -M main
git checkout -b develop
git checkout -b staging
git checkout main
git push --all origin
```

## 2. Server Preparation

These steps were performed on the Debian 13 server.

### 2.1. Connect via SSH

```bash
ssh -i ~/.ssh/id_ed25519_gaia root@72.62.125.163
```

### 2.2. Install Packages

Essential packages for hosting the application were installed.

```bash
apt-get update
apt-get install -y git nodejs npm nginx
```

## 3. Project Deployment

The project was cloned from GitHub into three separate directories.

### 3.1. Clone Repositories

From the `/var/www/` directory on the server:

```bash
# Navigate to the parent directory
cd /var/www

# Create a parent directory for the project
mkdir site02.com
cd site02.com

# Clone the repository into live, stage, and dev subdirectories
git clone <your-github-repository-url> live
git clone <your-github-repository-url> stage
git clone <your-github-repository-url> dev
```

### 3.2. Check Out Branches and Build

For each environment, the correct branch was checked out and the project was built. This process requires fetching from the remote first.

**Live Environment:**
```bash
cd /var/www/site02.com/live
git fetch origin
git checkout main
npm install
npm run build
```

**Staging Environment:**
```bash
cd /var/www/site02.com/stage
git fetch origin
git checkout stage
npm install
npm run build
```

**Development Environment:**
```bash
cd /var/www/site02.com/dev
git fetch origin
git checkout develop
npm install
npm run build
```

## 4. Nginx Configuration

Nginx was configured to serve the three environments on different ports.

### 4.1. Create Config Files

Three separate configuration files were created in `/etc/nginx/sites-available/`.

**Live (`site02-live`):**
```nginx
server {
    listen 3000;
    server_name _;

    root /var/www/site02.com/live/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

**Staging (`site02-staging`):**
```nginx
server {
    listen 3001;
    server_name _;

    root /var/www/site02.com/stage/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

**Development (`site02-dev`):**
```nginx
server {
    listen 3002;
    server_name _;

    root /var/www/site02.com/dev/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### 4.2. Enable Sites and Restart Nginx

The configurations were enabled by creating symbolic links, and Nginx was restarted.

```bash
# (Optional) Remove default site
rm /etc/nginx/sites-enabled/default

# Enable the new sites
ln -s /etc/nginx/sites-available/site02-live /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/site02-staging /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/site02-dev /etc/nginx/sites-enabled/

# Test configuration and restart
nginx -t
systemctl restart nginx
```

## 5. Firewall Configuration

The `ufw` firewall was installed and configured on the Debian server.

```bash
# Install ufw
apt-get install ufw

# IMPORTANT: Allow SSH traffic first
ufw allow ssh

# Allow application ports
ufw allow 3000/tcp
ufw allow 3001/tcp
ufw allow 3002/tcp

# Enable the firewall
ufw enable

# Check the status
ufw status
```

## 6. Final Access URLs

The sites are now accessible at the following URLs:

*   **Live:** `http://72.62.125.163:3000`
*   **Staging:** `http://72.62.125.163:3001`
*   **Development:** `http://72.62.125.163:3002`
