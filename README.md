# Local

## Deploiement

Alimenter les variables d'env:
- API_HOST_SUFFIX :
- FRONT_REVIEW_APP_NAME_PREFIX :

## Démarrer nginx en mode debug

Ajouter la directive de debug en haut du fichier `nginx.conf.erb`
```
error_log /var/log/nginx/error.log debug;
```

Démarrer nginx
``` shell
erb nginx.conf.erb > nginx.conf
docker run -v $(pwd)/nginxbase.conf:/etc/nginx/nginx.conf:ro -v $(pwd)/nginx.conf:/etc/nginx/conf.d/default.conf:ro --env-file .env -p 80:80 --entrypoint nginx-debug nginx '-g daemon off;' 2>&1 |egrep '^(Host: |X-Forwarded-Host: |.GET .* HTTP)'
```

Localiser une review-app active et récupérer le nom de l'application, ici `pix-bot-review-pr202`.

Exécuter cet appel
``` shell
curl -H "Host: bot-pr202.review.pix.fr" localhost:80/url
```

Vérifier les logs: le proxy doit être effectué vers `https://pix-bot-review-pr202.scalingo.io`.

```shell
"GET /url HTTP/1.0
X-Forwarded-Host: bot-pr202.review.pix.fr
Host: pix-bot-review-pr202.scalingo.io
```

> Pour les fronts du monorepo, comme la review app est commune à tous les fronts, une configuration spécifique
> est mise en place

Pour tester le proxy des fronts du monorepo exécuter ces appels
```shell
curl -H "Host: app-pr202.review.pix.fr" localhost:80/urlapp
curl -H "Host: orga-pr202.review.pix.fr" localhost:80/urlorga
```

Vérifier les logs: le proxy doit être effectué à chaque fois vers la review front
`https://pix-front-review-pr202.scalingo.io` avec un path préfixée par le nom de l'application.

```shell
"GET /app/urlapp HTTP/1.0
X-Forwarded-Host: app-pr202.review.pix.fr
Host: pix-front-review-pr202.scalingo.io
"GET /orga/urlorga HTTP/1.0
X-Forwarded-Host: orga-pr202.review.pix.fr
Host: pix-front-review-pr202.scalingo.io
```
