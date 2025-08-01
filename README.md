# Local

## Démarrer nginx en mode debug

Créer la conf nginx avec une directive de debug
``` shell
echo "error_log /var/log/nginx/error.log debug;" > nginx.conf
erb nginx.conf.erb >> nginx.conf
```
Démarrer le server nginx
```shell
docker compose up
```

Localiser une review-app active et récupérer le nom de l'application, ici `pix-orga-review-pr13072`.

Exécuter cet appel
``` shell
curl -H "Host: orga-pr13072.review.pix.fr" localhost:80/url &>/dev/null
```

Vérifier les logs: le proxy doit être effectué vers `https://pix-orga-review-pr13072.scalingo.io`.

```shell
docker logs pix-review-router 2>&1 | egrep '^(Host: |X-Forwarded-Host: |.* HTTP)'
```
Dans cet exemple, le résultat est:
```
2025/07/31 11:32:48 [debug] 11#11: *7 http request line: "GET /url HTTP/1.1"
"GET /url HTTP/1.0
X-Forwarded-Host: orga-pr13072.review.pix.fr
Host: pix-orga-review-pr13072.osc-fr1.scalingo.io
2025/07/31 11:32:48 [debug] 11#11: *7 HTTP/1.1 200 OK
```

