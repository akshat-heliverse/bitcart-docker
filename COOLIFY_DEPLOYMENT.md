# Bitcart Deployment on Coolify

This guide will help you deploy Bitcart on Coolify using Docker Compose.

## Quick Start

1. **Generate compose file** (set `BITCART_REVERSEPROXY=none`):

```bash
export BITCART_HOST=yourdomain.com
export BITCART_REVERSEPROXY=none
export BITCART_CRYPTOS=btc,xrg
./build.sh
```

2. **In Coolify**:

   - Create Docker Compose resource
   - Use `compose/generated.yml`
   - Add environment variables
   - Configure domains and ports

3. **Key Setting**: Always set `BITCART_REVERSEPROXY=none` - Coolify handles reverse proxy!

## Prerequisites

1. Coolify instance running
2. Domain name configured (or use Coolify's provided domain)
3. Git repository access (or upload files directly)

## Step 1: Prepare Environment Variables

In Coolify, you'll need to set these environment variables:

### Required Variables

```bash
BITCART_HOST=yourdomain.com  # Your domain or Coolify domain
BITCART_LETSENCRYPT_EMAIL=your@email.com  # For SSL certificates
```

### Optional but Recommended

```bash
# One domain mode (recommended for Coolify)
BITCART_ADMIN_API_URL=https://${BITCART_HOST}
BITCART_STORE_API_URL=https://${BITCART_HOST}

# Configure which cryptocurrencies to enable
BITCART_CRYPTOS=btc,xrg,eth  # Add coins you want (btc, xrg, eth, bnb, matic, trx, ltc, grs, xmr, bch)

# Network settings (defaults to mainnet)
XRG_NETWORK=mainnet  # or testnet
BTC_NETWORK=mainnet
ETH_NETWORK=mainnet

# Lightning Network (if needed)
BTC_LIGHTNING=false
LTC_LIGHTNING=false

# Reverse proxy - set to 'none' since Coolify handles this
BITCART_REVERSEPROXY=none
```

## Step 2: Generate Docker Compose File for Coolify

**Important**: Set `BITCART_REVERSEPROXY=none` - this automatically:

- Removes nginx, nginx-gen, nginx-https services
- Exposes ports directly (backend:8000, admin:4000, store:3000)
- Makes services ready for Coolify's reverse proxy

### Generate the Compose File

1. Clone the repository (if not already):

```bash
git clone https://github.com/bitcart/bitcart-docker
cd bitcart-docker
```

2. Set environment variables and generate:

```bash
export BITCART_HOST=yourdomain.com
export BITCART_REVERSEPROXY=none  # REQUIRED: Coolify handles reverse proxy
export BITCART_CRYPTOS=btc,xrg    # Your desired cryptocurrencies
export XRG_NETWORK=mainnet        # or testnet
export BTC_NETWORK=mainnet
./build.sh
```

3. The generated file will be at `compose/generated.yml` - this file is ready for Coolify!

**Note**: When `BITCART_REVERSEPROXY=none` is set, the generator automatically:

- Removes all nginx-related services
- Exposes ports directly (ports mapping instead of just expose)
- Removes VIRTUAL_HOST and LETSENCRYPT_HOST dependencies

## Step 4: Deploy in Coolify

1. **Create New Resource** in Coolify:

   - Choose "Docker Compose" as resource type
   - Connect your Git repository OR upload the compose file

2. **Configure the Application**:

   - Set the compose file path: `compose/generated.yml` (or upload directly)
   - Add all environment variables from Step 1

3. **Configure Ports** in Coolify:

   - **Backend**: Port 8000 (main API - configure as primary service)
   - **Admin**: Port 4000 (admin panel)
   - **Store**: Port 3000 (store frontend)
   - Database & Redis: Internal only (don't expose)

   **In Coolify**:

   - Set Backend (port 8000) as the main service
   - For Admin and Store, you can either:
     a) Use Coolify's path-based routing (recommended for one-domain mode)
     b) Create separate services with different domains

4. **Configure Domains**:

   - Add your domain in Coolify's domain settings
   - Coolify will handle SSL automatically

5. **Volumes**:
   - Ensure persistent volumes are configured:
     - `bitcart_datadir` - Main data directory
     - `backup_datadir` - Backups (external volume)
     - `dbdata` - Database data
     - `bitcoin_datadir` - Bitcoin daemon data
     - Other coin datadirs as needed

## Step 5: Configure Coolify Routing

Since you have multiple services (backend, admin, store), configure routing:

### Option A: One Domain Mode (Recommended)

Configure Coolify to route based on paths:

- Main domain → Backend (port 8000)
- `/admin` → Admin (port 4000)
- `/` → Store (port 3000) or Backend API

Set in environment:

```bash
BITCART_ADMIN_ROOTPATH=/admin
BITCART_STORE_ROOTPATH=/
BITCART_BACKEND_ROOTPATH=/api
```

### Option B: Separate Domains

- `api.yourdomain.com` → Backend (8000)
- `admin.yourdomain.com` → Admin (4000)
- `store.yourdomain.com` → Store (3000)

## Step 6: Post-Deployment

1. **Access your instance**:

   - Store: `https://yourdomain.com` (or configured path)
   - Admin: `https://yourdomain.com/admin` (or configured path)
   - API: `https://yourdomain.com/api` (or configured path)

2. **Initial Setup**:
   - Access admin panel
   - Create your first store
   - Configure payment methods

## Troubleshooting

### Invalid Volume Definition Error

**Error**: `Invalid volume target: contains forbidden character '${' (variable substitution with potential command injection)`

**Solution**: Coolify doesn't allow variable substitution in volume paths for security. The compose file has been fixed to remove SSH authorized_keys volume mounts. If you regenerate the file, comment out these lines:

```yaml
# Remove or comment out in backend and worker services:
# - /root/.ssh/authorized_keys:${BITCART_SSH_AUTHORIZED_KEYS}
```

SSH host access is not needed for Coolify deployments, so this volume mount can be safely removed.

### Services not starting

- Check environment variables are set correctly
- Verify volumes are mounted properly
- Check logs in Coolify dashboard

### Database connection issues

- Ensure database service is running
- Check `DB_HOST=database` is correct (service name)

### Port conflicts

- Ensure ports aren't already in use
- Check Coolify port mappings

### Reverse proxy issues

- If using Coolify's reverse proxy, set `BITCART_REVERSEPROXY=none`
- Remove nginx-related services from compose file

## Notes

- Coolify handles SSL/TLS automatically, so you don't need nginx-https
- Use Coolify's built-in reverse proxy instead of nginx
- Persistent volumes are important for data retention
- Database and Redis should be internal-only (not exposed)

## Example Minimal Configuration

For a quick start with just Bitcoin and Ergon:

```bash
BITCART_HOST=bitcart.yourdomain.com
BITCART_LETSENCRYPT_EMAIL=admin@yourdomain.com
BITCART_CRYPTOS=btc,xrg
BITCART_REVERSEPROXY=none
XRG_NETWORK=mainnet
BTC_NETWORK=mainnet
BITCART_ADMIN_API_URL=https://${BITCART_HOST}
BITCART_STORE_API_URL=https://${BITCART_HOST}
```
