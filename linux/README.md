## Non-root service can't access certbot's certificate located at `/etc/letsencrypt/live`

Make sure that service runs in its own group, i.e.: headscale will run as group `headscale`, then run the following command:

```
chmod 0755 /etc/letsencrypt/{live,archive}
chgrp <group> /etc/letsencrypt/live/$domain/{fullchain.pem,privkey.pem}
chmod 0640 /etc/letsencrypt/live/$domain/{fullchain.pem,privkey.pem}
```

> For historical reasons, the containing directories are created with permissions of `0700` meaning that certificates are accessible only to servers that run as the root user. If you will never downgrade to an older version of Certbot, then you can safely fix this using `chmod 0755 /etc/letsencrypt/{live,archive}`.
>
> For servers that drop root privileges before attempting to read the private key file, you will also need to use `chgrp` and `chmod 0640` to allow the server to read `/etc/letsencrypt/live/$domain/privkey.pem`.

Source: https://eff-certbot.readthedocs.io/en/latest/using.html#where-are-my-certificates
