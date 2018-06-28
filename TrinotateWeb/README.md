# TrinotateWeb in a Docker container

Container is self contained, data is baked in.

Requires:

* TrinotateAnno.sqlite - generated via Trinotate
* lighttpd.conf (provided) - preconfigured, don't edit.

```bash
docker build -t trinotate:dataset_name .
```

## Running on port 4569 for testing:

```bash
docker run --name DatasetName_Trinotate --rm -it -d -p 4569:80 trinotate:dataset_name
```

## Production on port 4569:
```bash
docker run --name DatasetName_Trinotate --restart=always -it -d -p 4569:80 trinotate:dataset_name
```

You should see the app running if you point your browser (or `curl`) to:

```bash
curl "http://localhost:4569/cgi-bin/index.cgi?sqlite_db=/data/TrinotateAnno.sqlite"
```

Use Apache to forward (proxy) a nice URL (eg /apps/DatasetName_Trinotate) to the container host, port 4569, behind `.htaccess` (as per other containerised services proxied in `bio-web-ansible`).

## Option: external data and config in a host directory

With a few small edits to the Dockerfile (comment out Trinotate download and sqlite db COPY), you can instead use an external copy of Trinotate and a database on the host.
You might want this for data that is going to be in flux for a while, before baking it permanently in a container (?).

```bash
docker run --name DatasetName_Trinotate --rm -it -d -p 4569:80 -v $(pwd):/app -v /home/sarah.williams/bin/Trinotate-Trinotate-v3.1.1/:/app/Trinotate -v $(pwd)/TrinotateAnno.sqlite:/data/TrinotateAnno.sqlite trinotate:dataset_name
```

## Apache config

Use this to forward incoming requests to `/apps/trinotate/DatasetName/` -> the port on the Docker container (4569), with a custom htaccess file for Basic Auth.

TrinotateWeb makes requests to https://canvasxpress.org/, but the certificates currently appear invalid (self signed or out of date).
The user should visit https://canvasxpress.org/ first and accept the insecure certificate so that icons in TrinotateWeb load correctly.

```apache2
    # /apps/trinotate/DatasetName
    <Proxy "http://localhost:4569/*">
      Order deny,allow
      Allow from all
      Authtype Basic
      Authname "Restricted Content"
      AuthUserFile /etc/apache2/htaccess/DatasetName
      Require valid-user
    </Proxy>
    
    RewriteEngine on
    
    # For TrinotateWeb inside a Docker container - absolute URLs mean /css and /js links break
    # when proxied, unless we use this RewriteCond trick detecting referrer. 
    RewriteCond "%{HTTP_REFERER}" ".*bioinformatics.erc.monash.edu(?:.au)?/apps/trinotate/DatasetName/.*" [NV]
    RewriteRule ^/css/(.*)$ "http://localhost:4569/css/$1" [P]
    RewriteCond "%{HTTP_REFERER}" ".*bioinformatics.erc.monash.edu(?:.au)?/apps/trinotate/DatasetName/.*" [NV]
    RewriteRule ^/js/(.*)$ "http://localhost:4569/js/$1" [P]
    RewriteRule ^/apps/trinotate/DatasetName$ /apps/trinotate/DatasetName/ [R]
    RewriteRule ^/apps/trinotate/DatasetName/(.*)$ "http://localhost:4569/$1" [P]
```
