# Nextcloud Talk: How to Set Up a TURN Server for Better Call Quality

To set up a TURN server for Nextcloud Talk, you'll need to configure a TURN server to ensure reliable media streaming, especially in peer-to-peer calls when direct connections fail due to NAT (Network Address Translation) or firewall issues. Here's a step-by-step guide on how to set up a TURN server using **coturn**:

## Step 1: Install coturn

First, install the `coturn` package on your server:

```bash
sudo apt install coturn
```

## Step 2: Configure coturn

Once installed, configure coturn by editing the configuration file at `/etc/turnserver.conf`. The configuration should look something like this:

```bash
listening-port=3478
fingerprint
lt-cred-mech
use-auth-secret
static-auth-secret=YOUR_SECRET_KEY
realm=yourdomain.com
total-quota=100
bps-capacity=0
stale-nonce
no-loopback-peers
no-multicast-peers
```

**Key fields**:

- `listening-port`: Port for the TURN server (default is `3478`).
- `use-auth-secret`: Enable the use of shared secrets.
- `static-auth-secret`: This is the secret key you will use to authenticate Nextcloud with the TURN server. Generate it using a tool like OpenSSL:

  ```bash
  openssl rand -hex 32
  ```

  Replace `YOUR_SECRET_KEY` with the key generated.

- `realm`: Set it to your domain name (e.g., `yourdomain.com`).

## Step 3: Enable and Start coturn

Now enable and start the TURN server:

```bash
sudo systemctl enable coturn
sudo systemctl start coturn
```

## Step 4: Open Ports on Firewall

Ensure the necessary ports are open on your firewall. By default, TURN uses port `3478` for both UDP and TCP traffic, so allow these in your firewall rules:

```bash
sudo ufw allow 3478/tcp
sudo ufw allow 3478/udp
```

If you're using TLS for TURN (recommended for security), you might also open port `5349`:

```bash
sudo ufw allow 5349/tcp
```

## Step 5: Configure Nextcloud Talk to Use TURN

1. Go to **Nextcloud Settings** > **Talk**.
2. In the **TURN server configuration**, add your TURN server details:
   - **TURN server URL**: `turn:yourdomain.com:3478?transport=udp`
   - **TURN server secret**: Add the same secret key (`YOUR_SECRET_KEY`) from the `turnserver.conf`.
   - **TURN server username**: Leave this empty, since you're using secret-based authentication.
   - **TURN server password**: Leave this empty as well.

## Step 6: Testing and Troubleshooting

To verify the setup, make a video call in Nextcloud Talk. If it works smoothly (especially in different network environments), the TURN server is functioning correctly.

Check logs for any issues:

- TURN server logs: `/var/log/turnserver.log`
- Nextcloud logs: `/var/snap/nextcloud/current/logs/nextcloud.log` (for Snap installation).

With this setup, your Nextcloud Talk should have a working TURN server for handling peer-to-peer connection failures and ensuring smooth video/audio communication.
