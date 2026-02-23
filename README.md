# Multi-Repository GitHub Origin Masking

**Domain:** repos.devopsnerds.com  
**Objective:** Mask multiple GitHub repositories behind single custom domain using path-based routing

## Repositories

- `/aws-creds-manager` → https://github.com/aafaq-rashid-comprinno/aws-creds-manager
- `/mongo` → https://github.com/aafaq-rashid-comprinno/mongo

## Setup

1. Add to /etc/hosts:
```bash
sudo sh -c 'echo "127.0.0.1 repos.devopsnerds.com" >> /etc/hosts'
```

2. Generate SSL certificate:
```bash
mkdir -p ssl && openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout ssl/nginx.key -out ssl/nginx.crt \
  -subj "/CN=repos.devopsnerds.com"
```

3. Run:
```bash
docker build -t multi-repo-proxy .
docker run -d -p 9080:80 -p 9443:443 --name multi-repo-proxy multi-repo-proxy
```

## Access

**Web Browser:**
- https://repos.devopsnerds.com:9443/aws-creds-manager
- https://repos.devopsnerds.com:9443/mongo

**Git Clone:**
```bash
git -c http.sslVerify=false clone https://repos.devopsnerds.com:9443/aws-creds-manager
git -c http.sslVerify=false clone https://repos.devopsnerds.com:9443/mongo
```

(Self-signed certificate - disable SSL verification for testing)

## Add New Repository

1. Edit `nginx.conf` and add new location block:
```nginx
location /new-repo {
    proxy_pass https://github.com/username/new-repo;
    proxy_ssl_server_name on;
    proxy_set_header Host github.com;
    proxy_set_header X-Forwarded-Proto https;
    
    sub_filter 'github.com/username/new-repo' 'repos.devopsnerds.com/new-repo';
    sub_filter 'github.com' 'repos.devopsnerds.com';
    sub_filter_once off;
    sub_filter_types *;
}
```

2. Update the root location to list the new repo:
```nginx
location = / {
    return 200 "Available repos:\n- /aws-creds-manager\n- /mongo\n- /new-repo\n";
    add_header Content-Type text/plain;
}
```

3. Rebuild and restart:
```bash
docker rm -f multi-repo-proxy
docker build -t multi-repo-proxy .
docker run -d -p 9080:80 -p 9443:443 --name multi-repo-proxy multi-repo-proxy
```

## Validation

- ✅ Multiple repos accessible via single domain
- ✅ Path-based routing (/repo-name)
- ✅ Git clone works for all repos
- ✅ No GitHub references visible

## Files

- `Dockerfile` - Container configuration
- `nginx.conf` - Nginx reverse proxy with multiple location blocks
- `ssl/` - SSL certificates (self-signed)
