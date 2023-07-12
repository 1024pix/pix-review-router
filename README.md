# Local

## Configuration

La configuration se fait par les variables d'environnement suivantes.
```dotenv
# Suffix of the api host
# obligatoire: oui
# type: String
# valeur par défaut: aucune
# exemple: osc-fr1.scalingo.io
API_HOST_SUFFIX=

# Prefix name for the front review apps
# obligatoire: non
# type: String
# valeur par défaut: pix
# exemple: custom-domain
FRONT_REVIEW_APP_NAME_PREFIX=

# URL du buildpack nginx Scalingo
# obligatoire: non
# type: String
# valeur par défaut: aucune
# exemple: pix-4pix
BUILDPACK_URL=https://github.com/Scalingo/nginx-buildpack
```

## Démarrer nginx en mode debug

Ajouter la directive de debug en haut du fichier `nginx.conf.erb`
```
error_log /var/log/nginx/error.log debug;
```

Démarrer nginx
```shell
  export FRONT_REVIEW_APP_NAME_PREFIX=pix
  export API_HOST_SUFFIX=osc-fr1.scalingo.io
  erb nginx.conf.erb > nginx.conf
  docker rm review-router
  docker run \
      -v $(pwd)/nginxbase.conf:/etc/nginx/nginx.conf:ro \
      -v $(pwd)/nginx.conf:/etc/nginx/conf.d/default.conf:ro \
      -p 80:80 \
      --entrypoint nginx-debug \
      --name="review-router" \
      nginx '-g daemon off;' 2>&1 \
       |egrep '^(Host: |X-Forwarded-Host: |.GET .* HTTP )'
```
La console doit se mettre en attente.

Vérifier les logs du container (dans un autre terminal)
```
  docker logs review-router
```

Localiser une review-app active et récupérer le nom de l'application, ici `pix-bot-review-pr202`.

Exécuter cet appel
```shell
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

Pour tester le proxy des fronts du monorepo, exécuter ces appels.

Si vous utilisez les variables par défaut
```shell
curl -H "Host: app-pr202.review.pix.fr" localhost:80/connexion
curl -H "Host: orga-pr202.review.pix.fr" localhost:80/campagnes/les-miennes
```

En alimentant la variable d'env `FRONT_REVIEW_APP_NAME_PREFIX=custom`
```shell
curl -H "Host: app-pr202.review.pix.custom.domain" localhost:80/connexion
curl -H "Host: orga-pr202.review.pix.custom.domain" localhost:80/campagnes/les-miennes
```

Vérifier les logs : le proxy doit être effectué à chaque fois vers la review front
`https://pix-front-review-pr202.scalingo.io` avec un path préfixée par le nom de l'application.

```shell
"GET /app/urlapp HTTP/1.0
X-Forwarded-Host: app-pr202.review.pix.fr
Host: pix-front-review-pr202.scalingo.io
"GET /orga/urlorga HTTP/1.0
X-Forwarded-Host: orga-pr202.review.pix.fr
Host: pix-front-review-pr202.scalingo.io
```
