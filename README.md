# Multi-Repository GitHub Origin Masking

**Domain:** vcs.hivepro.com  
**Objective:** Automatically expose ALL repositories from a GitHub user/organization behind a custom domain

## Dynamic Routing

All repositories from `readonlyuserhivepro` are automatically accessible:
- `https://vcs.hivepro.com/<any-repo-name>`

No manual configuration needed per repository!

## Setup

1. Add to /etc/hosts:
```bash
sudo sh -c 'echo "127.0.0.1 vcs.hivepro.com" >> /etc/hosts'
```

2. Generate SSL certificate (if not already present):
```bash
mkdir -p ssl && openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout ssl/nginx.key -out ssl/nginx.crt \
  -subj "/CN=vcs.hivepro.com"
```

3. Run:
```bash
docker-compose up -d
```

Or without docker-compose:
```bash
docker build -t multi-repo-proxy .
docker run -d -p 9080:80 -p 9443:443 --name multi-repo-proxy multi-repo-proxy
```

## Access

**Important:** GitHub requires authentication for git operations over HTTPS, even for public repositories. You need to provide credentials when cloning.

**Git Clone with Authentication:**
```bash
# Option 1: Inline credentials (not recommended for production)
git -c http.sslVerify=false clone https://USERNAME:TOKEN@vcs.hivepro.com:9443/<repo-name>.git

# Option 2: Use credential helper (recommended)
git config --global credential.helper store
git -c http.sslVerify=false clone https://vcs.hivepro.com:9443/<repo-name>.git
# Enter your GitHub username and personal access token when prompted
```

**Web Browser:**
- https://vcs.hivepro.com:9443/<any-repo-name>

Examples:
```bash
# With GitHub personal access token
git -c http.sslVerify=false clone https://your-github-username:ghp_yourtoken@vcs.hivepro.com:9443/repo-name.git
```

## Change GitHub User/Organization

Edit `nginx.conf` and change the username in the proxy_pass line:
```nginx
proxy_pass https://github.com/YOUR-USERNAME/$1$is_args$args;
```

And update the sub_filter:
```nginx
sub_filter 'github.com/YOUR-USERNAME/' 'vcs.hivepro.com/';
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
