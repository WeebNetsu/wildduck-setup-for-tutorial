# WildDuck Base For Tutorials

Based on: https://github.com/nodemailer/wildduck-dockerized

**Note this setup should not be used in production. It is a basic setup with NO additional security layers and is not a fully running implementation. This is strictly for following tutorials.**

## 1. Clone and create `.env`

```bash
git clone https://github.com/WeebNetsu/wildduck-setup-for-tutorial.git
cd wildduck-setup-for-tutorial
cp example.env .env
```

## 2. Edit `.env` (optional)

Change these values and leave the rest alone **if you want to use your own domain**. I do not recommend changing this if you are not sure what you're doing.

```env
PUBLIC_HOSTNAME=mail.teacher.com
MAIL_DOMAIN=teacher.com

WILDDUCK_API_ACCESS_TOKEN=local-api-token
WILDDUCK_API_ACCESSCONTROL_SECRET=local-access-secret
WILDDUCK_LOOP_SECRET=local-loop-secret
WILDDUCK_DKIM_SECRET=local-dkim-secret

TRAEFIK_TLS_MODE=file
TRAEFIK_CERT_FILE=./certs/teacher.pem
TRAEFIK_KEY_FILE=./certs/teacher-key.pem
KIRIN_TLS_CERT_FILE=./certs/teacher.pem
KIRIN_TLS_KEY_FILE=./certs/teacher-key.pem
```

## 3. Make a certificate for `mail.teacher.com`

If you chose your own domain, be sure to update this command before running it.

Your mail client and browser would complain about a hostname mismatch. Generate your own with [mkcert](https://github.com/FiloSottile/mkcert):

```bash
mkcert -install
mkcert -cert-file certs/teacher.pem -key-file certs/teacher-key.pem mail.teacher.com teacher.com localhost
chmod 644 certs/teacher-key.pem   # Kirin runs as UID 1000 and must be able to read it
```

`mkcert -install` makes your own machine trust the certificate.

## 4. Edit `docker-compose.yml`

In the `zonemta` service, add these two lines to the end of the `environment:` block:

```yaml
APPCONF_zones_default_host: traefik
APPCONF_zones_default_port: "25"
```

## 5. Map the hostname on your computer

Add this line to `/etc/hosts`

```
127.0.0.1 mail.teacher.com
```

Don't map `teacher.com` itself to `127.0.0.1`.

If you're on NixOS:

```nix
networking.hosts = {
    # for wildduck
    "127.0.0.1" = [ "mail.teacher.com" ];
};
```

## 6. Start the stack

```bash
docker compose up -d --build
docker compose ps    # everything should be running/healthy
```

Ports 25, 80, 443, 465, 993 and 995 must be free on your machine.
