# Local

## Démarrer nginx en mode debug

Démarrer nginx avec une directive de debug
``` shell
echo "error_log /var/log/nginx/error.log debug;" > nginx.conf
erb nginx.conf.erb >> nginx.conf
docker run -v $(pwd)/nginxbase.conf:/etc/nginx/nginx.conf:ro -v $(pwd)/nginx.conf:/etc/nginx/conf.d/default.conf:ro -p 80:80 --entrypoint nginx-debug nginx '-g daemon off;' 2>&1 |egrep '^(Host: |X-Forwarded-Host: |.GET .* HTTP)'
```

Localiser une review-app active et récupérer le nom de l'application, ici `pix-app-review-pr202`.

Exécuter cet appel
``` shell
curl -H "Host: app-pr202.review.pix.fr" localhost:80/url
```

Vérifier les logs: le proxy doit être effectué vers `https://pix-app-review-pr202.scalingo.io`.

```shell
"GET /url HTTP/1.0
X-Forwarded-Host: app-pr202.review.pix.fr
Host: pix-app-review-pr202.scalingo.io
```
