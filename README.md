# DocumentacionDockerNgnix-Proxy
Documentacion de como he instalado docker y Ngnix proxy en un server debian,
esto nos servira para enlazar directamente nuestros docker con una url


## Pasos
- Instalar Docker
- Desplegar docker ngnix-proxy

## Instalar Docker
Para instlar Docker, seguir la pagina oficial
[Install Docker debian](https://docs.docker.com/engine/install/debian/)
He ejecutado estos comandos en el server
```bash
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

 echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
## Desplegar docker ngnix-proxy
Una vez tenemos Docker funcionando
Desplegar el ngnix-proxy con acme-companion para que tenga certificados validos para https
https://github.com/nginx-proxy/acme-companion

Ejecute estos comandos en mi servidor
```bash
 docker run --detach \
    --name nginx-proxy \
    --publish 80:80 \
    --publish 443:443 \
    --volume certs:/etc/nginx/certs \
    --volume vhost:/etc/nginx/vhost.d \
    --volume html:/usr/share/nginx/html \
    --volume /var/run/docker.sock:/tmp/docker.sock:ro \
    nginxproxy/nginx-proxy


    docker run --detach \
    --name nginx-proxy-acme \
    --volumes-from nginx-proxy \
    --volume /var/run/docker.sock:/var/run/docker.sock:ro \
    --volume acme:/etc/acme.sh \
    --env "DEFAULT_EMAIL=mail@yourdomain.tld" \
    nginxproxy/acme-companion
```

Para Probar que funcionaba como esperaba  probe con estos  dos servicios, primero con el del ejemplo, grafana, y luego con netdata

```
docker run --detach \
    --name grafana \
    --env "VIRTUAL_HOST=othersubdomain.yourdomain.tld" \
    --env "VIRTUAL_PORT=3000" \
    --env "LETSENCRYPT_HOST=othersubdomain.yourdomain.tld" \
    --env "LETSENCRYPT_EMAIL=mail@yourdomain.tld" \
    grafana/grafana


docker run -d --name=netdata \
  --env "VIRTUAL_HOST=othersubdomain.yourdomain.tld" \
  --env "VIRTUAL_PORT=19999" \
  --env "LETSENCRYPT_HOST=othersubdomain.yourdomain.tld" \
  --env "LETSENCRYPT_EMAIL=mail@gmail.com" \
  -v $(pwd)/netdataconfig/netdata:/etc/netdata:ro \
  -v netdatalib:/var/lib/netdata \
  -v netdatacache:/var/cache/netdata \
  -v /etc/passwd:/host/etc/passwd:ro \
  -v /etc/group:/host/etc/group:ro \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /etc/os-release:/host/etc/os-release:ro \
  --restart unless-stopped \
  --cap-add SYS_PTRACE \
  --security-opt apparmor=unconfined \
  netdata/netdata
```
Dejamos pendiente para mas adelante jugar con los path, aunque aqui hay  una pregunta en stackOverflow que  parece que nos puede ayudar

https://stackoverflow.com/questions/39514293/docker-nginx-proxy-how-to-route-traffic-to-different-container-using-path-and-n#