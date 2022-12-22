# Local

## Démarrer nginx en mode debug

Ajouter la directive de debug en haut du fichier `nginx.conf.erb`
```
error_log /var/log/nginx/error.log debug;
```

Démarrer nginx
``` shell
erb nginx.conf.erb > nginx.conf
docker run -v $(pwd)/nginx.conf:/etc/nginx/conf.d/default.conf:ro -p 80:80 --entrypoint nginx-debug nginx '-g daemon off;'
```

Localiser une review-app active et récupérer le nom de l'application, ici `pix-bot-review-pr202`.

Exécuter cet appel
``` shell
curl -H "Host: pix-bot-review-pr202.review.pix.fr" localhost:80/pix-bot-review-pr202
```

Vérifier les logs: le proxy doit être effectué vers `https://pix-bot-review-pr202.osc-fr1.scalingo.io`.

```shell
"GET /pix/pix-bot-review-pr202 HTTP/1.0
X-Forwarded-Host: pix-bot-review-pr202.review.pix.fr
X-Forwarded-For: 172.17.0.1
Pix-Application: pix
Host: pix-pix-review-bot-review-pr202.scalingo.io
Connection: close
User-Agent: curl/7.81.0
Accept: */*
```
