# Certificate

The certificate should have the FQDN equals to application ingress host (like a valid certificate...):

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout portus.key -out portus.cert -subj "/CN=portus.192-168-99-103.nip.io"
```

## Verifying

You can verify you certificate with:

```bash
openssl s_client -connect portus.192-168-99-103.nip.io:443
```

Just scroll up and look for `subject`.
