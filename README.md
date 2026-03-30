# FitRehab — Exercise & Rehabilitation Planner

A web application that helps users discover physical exercises based on target muscle groups and available equipment. Built for rehabilitation planning and general fitness, powered by the ExerciseDB API.

---

## Purpose

FitRehab solves a real problem: finding the right exercises when you have limited equipment or specific rehabilitation needs. Users can filter thousands of exercises by muscle group (e.g., "Back", "Biceps") and equipment (e.g., "Dumbbell", "Body Weight"), then view animated GIFs demonstrating proper form.
---

## Features

- **Filter by Muscle Group** — 15+ target muscle groups (Back, Chest, Biceps, etc.)
- **Filter by Equipment** — From dumbbells to body weight to resistance bands
- **Combined Filtering** — Select both muscle + equipment simultaneously
- **Live Search** — Search within results by exercise name
- **Sort Controls** — Sort alphabetically, by muscle, or by equipment
- **Exercise Detail Modal** — View animated GIF, secondary muscles, body part
- **Pagination** — Navigate large result sets cleanly
- **Quick Select Chips** — Browse by muscle group with one click
- **Error Handling** — Graceful messages for API failures or empty results
- **Responsive Design** — Works on desktop and mobile

---

## Demo Video 
 **This is the YouTube video to understand how Fitreh works: https://www.youtube.com/watch?v=W1leKypLEbo
 
---

## Tech Stack

- **Frontend**: Pure HTML, CSS, JavaScript (no frameworks)
- **API**: [ExerciseDB on RapidAPI](https://rapidapi.com/justin-WFnsXH_t6/api/exercisedb)
- **Server**: Nginx on Ubuntu 16.04
- **Load Balancer**: HAProxy

---

## 📦 Running Locally

Clone the repository and open the file directly in your browser — no build step needed:

```bash
git clone https://github.com/<your-username>/fitrehab.git
cd fitrehab
# Open index.html in your browser
open index.html        # macOS
xdg-open index.html   # Linux
```

Or serve with Python for a local server:

```bash
python3 -m http.server 8080
# Then visit http://localhost:8080
```

> **API Key**: The RapidAPI key is included in the source. For production, move this to a backend proxy to keep it secret.

---

## ☁️ Deployment

The application is deployed on two web servers behind an HAProxy load balancer.

### Server Information

| Name   | IP              | Role          |
|--------|-----------------|---------------|
| web-01 | 35.173.178.76   | Web Server 1  |
| web-02 | 54.174.21.203   | Web Server 2  |
| lb-01  | 54.211.192.50   | Load Balancer |

**Live URL**: https://www.ibenito.tech

---

### Step 1 — Deploy to Web Servers

SSH into each server and run the following:

```bash
# Create web root directory
sudo mkdir -p /var/www/fitrehab

# Copy the app file
sudo cp index.html /var/www/fitrehab/index.html

# Install nginx
sudo apt-get update && sudo apt-get install -y nginx
```

Create the nginx config at `/etc/nginx/sites-available/fitrehab`:

```nginx
server {
    listen 80;
    server_name _;

    root /var/www/fitrehab;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
        expires 7d;
        add_header Cache-Control "public, immutable";
    }

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
}
```

Enable it:

```bash
sudo ln -sf /etc/nginx/sites-available/fitrehab /etc/nginx/sites-enabled/fitrehab
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl reload nginx
```

Repeat this on **both web-01 and web-02**.

---

### Step 2 — Load Balancer (HAProxy)

The HAProxy configuration on lb-01 (`/etc/haproxy/haproxy.cfg`) distributes traffic between both web servers and handles SSL termination:

```haproxy
frontend www-http
    bind *:80
    http-request redirect scheme https code 301

frontend www-https
    bind *:443 ssl crt /etc/haproxy/certs/www.ibenito.tech.pem
    option forwardfor
    http-request set-header X-Forwarded-Proto https
    default_backend www-backend

backend www-backend
    balance roundrobin
    server web-01 35.173.178.76:80 check
    server web-02 54.174.21.203:80 check
```

This ensures:
- HTTP traffic is redirected to HTTPS (301)
- SSL is terminated at the load balancer
- Requests are distributed round-robin between web-01 and web-02

---

### Using the Deploy Script

Alternatively, use the included script from your local machine:

```bash
chmod +x deploy.sh
./deploy.sh
```

This automatically copies the app to both servers and configures nginx.

---

## 🔌 API Reference

**API**: ExerciseDB  
**Provider**: RapidAPI  
**Documentation**: https://rapidapi.com/justin-WFnsXH_t6/api/exercisedb

### Endpoints Used

| Endpoint | Description |
|----------|-------------|
| `GET /exercises/targetList` | List of all target muscle groups |
| `GET /exercises/equipmentList` | List of all equipment types |
| `GET /exercises/target/{muscle}` | Exercises by target muscle |
| `GET /exercises/equipment/{type}` | Exercises by equipment |
| `GET /exercises` | All exercises (with pagination) |

---

## ⚠️ Challenges & Solutions

1. **Combined filtering**: The API doesn't support filtering by both muscle AND equipment simultaneously. Solved by fetching by primary filter (muscle) then filtering client-side by equipment.

2. **API rate limits**: Added `limit=200` to each request to minimize total API calls while keeping results comprehensive.

3. **GIF loading performance**: Used `loading="lazy"` on all exercise GIF images to prevent slow initial page load.

4. **SSL on load balancer**: Combined `fullchain.pem` and `privkey.pem` into a single `.pem` file for HAProxy SSL termination.

---

## 📝 Credits

- Exercise data and GIFs: [ExerciseDB](https://rapidapi.com/justin-WFnsXH_t6/api/exercisedb) by yuhonas on RapidAPI
- Fonts: [Google Fonts](https://fonts.google.com) (Bebas Neue, DM Sans, Space Mono)
- Icons: Unicode emoji

---

## 🔒 Security Note

The API key is currently embedded in the frontend JavaScript. For a production application, this should be proxied through a backend server (Node.js/Python) to prevent public exposure of the key.

---

## 📄 License

MIT
