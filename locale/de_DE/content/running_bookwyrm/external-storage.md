- - -
Title: Externer Speicher Date: 2021-06-07 Order: 5
- - -

Standardmäßig verwendet BookWyrm lokalen Speicher für statische Assets (Favicon, Standard-Avatar, etc.) und Medien (Benutzer-Avatare, Buchtitelbilder usw.), aber Sie können einen externen Speicherdienst verwenden, um diese Dateien zu bereitzustellen. BookWyrm verwendet `django-storages`, um externen Speicher wie S3-kompatible Dienste, Apache Libcloud oder SFTP anzubinden.

## S3-kompatibler Speicher

### Einrichtung

Erstellen Sie einen Bucket in Ihrem S3-kompatiblen Dienst der Wahl, zusammen mit einer Zugangsschlüssel-ID und einem geheimen Zugriffsschlüssel. Diese können selbst gehostet sein, wie [Ceph](https://ceph.io/en/) (LGPL 2.1/3.0) oder [MinIO](https://min.io/) (GNU AGPL v3.0) oder kommerziell ([Scaleway](https://www.scaleway.com/en/docs/object-storage-feature/), [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-create-a-digitalocean-space-and-api-key)…).

Diese Anleitung wurde mit Scaleway Object Storage getestet. Wenn Sie einen anderen Dienst verwenden, teilen Sie bitte Ihre Erfahrungen (insbesondere wenn Sie andere Schritte unternehmen mussten), indem Sie einen Problembericht im [BookWyrm Dokumentations](https://github.com/bookwyrm-social/documentation)-Repository einreichen.

### Was erwartet Sie

Wenn Sie eine neue BookWyrm-Instanz starten, wird der Prozess sein:

- Richten Sie Ihren externen Speicherservice ein
- Aktiviere externen Speicher auf BookWyrm
- Starten Sie Ihre BookWyrm-Instanz
- Instanz Konnektor aktualisieren

Wenn Sie Ihre Instanz bereits gestartet haben und Bilder auf den lokalen Speicher hochgeladen wurden, wird der Prozess sein:

- Richten Sie Ihren externen Speicherdienst ein
- Kopieren Sie Ihre lokalen Medien auf externen Speicher
- Aktiviere externen Speicher auf BookWyrm
- Starten Sie Ihre BookWyrm-Instanz neu
- Instanz-Konnektor aktualisieren

### BookWyrm-Einstellungen

Bearbeiten Sie Ihre `.env`-Datei, indem Sie die folgenden Zeilen auskommentieren:

- `AWS_ACCESS_KEY_ID`: Ihre Zugangsschlüssel-ID
- `AWS_SECRET_ACCESS_KEY`: Ihr geheimer Zugangsschlüssel
- `AWS_STORAGE_BUCKET_NAME`: Ihr Bucket Name
- `AWS_S3_REGION_NAME`: z.B. `"eu-west-1"` für AWS, `"fr-par"` für Scaleway oder `"nyc3"` für Digital Ocean

Wenn Ihr S3-kompatibler Dienst Amazon AWS ist, sollten Sie startklar sein. Wenn nicht, müssen Sie die folgenden Zeilen wieder kommentieren:

- `AWS_S3_CUSTOM_DOMAIN`: die Domain, die die Assets bereitstellen soll, z.B. `"example-bucket-name.s3.fr-par.scw.cloud"` oder `"${AWS_STORAGE_BUCKET_NAME}.${AWS_S3_REGION_NAME}.digitaloceanspaces.com"`
- `AWS_S3_ENDPOINT_URL`: der S3-API-Endpunkt, z.B. `"https://s3.fr-par.scw.cloud"` oder `"https://${AWS_S3_REGION_NAME}.digitaloceanspaces.com"`

### Kopieren lokaler Medien auf externen Speicher

Wenn Ihre BookWyrm-Instanz bereits läuft und Medien hochgeladen wurden (Benutzer-Avatare, Buchtitelbilder…), müssen Sie hochgeladene Medien in Ihren Bucket migrieren.

Diese Aufgabe wird mit dem Befehl erledigt:

```bash
./bw-dev copy_media_to_s3
```

### Aktivierung des externen Speichers für BookWyrm

To enable the S3-compatible external storage, you will have to edit your `.env` file by changing the property value for `USE_S3` from `false` to `true`:

```
USE_S3=true
```

If your external storage is being served over HTTPS (which most are these days), you'll also need to make sure that `USE_HTTPS` is set to `true`, so images will be loaded over HTTPS:

```
USE_HTTPS=true
```

#### Statische Assets

Then, you will need to run the following command, to copy the static assets to your S3 bucket:

```bash
./bw-dev collectstatic
```

#### CORS-Einstellungen

Once the static assets are collected, you will need to set up CORS for your bucket.

Some services like Digital Ocean provide an interface to set it up, see [Digital Ocean doc: How to Configure CORS](https://docs.digitalocean.com/products/spaces/how-to/configure-cors/).

If your service doesn’t provide an interface, you can still set up CORS with the command line.

Create a file called `cors.json`, with the following content:

```json
{
  "CORSRules": [
    {
      "AllowedOrigins": ["https://MY_DOMAIN_NAME", "https://www.MY_DOMAIN_NAME"],
      "AllowedHeaders": ["*"],
      "AllowedMethods": ["GET", "HEAD", "POST", "PUT", "DELETE"],
      "MaxAgeSeconds": 3000,
      "ExposeHeaders": ["Etag"]
    }
  ]
}
```

Replace `MY_DOMAIN_NAME` with the domain name(s) of your instance.

Then, run the following command:

```bash
./bw-dev set_cors_to_s3 cors.json
```

Keine Ausgabe bedeutet, dass es gut sein sollte.

Wenn Sie eine neue BookWyrm-Instanz starten, können Sie sofort zu den Installationsanweisungen zurückkehren. Wenn nicht, lesen Sie weiter.

### Aktualisiere deine Instanz

Once the media migration has been done and the static assets are collected, you can load the new `.env` configuration and restart your instance with:

```bash
./bw-dev up -d
```

If all goes well, your storage has been changed without server downtime. If some fonts are missing (and your browser’s JS console lights up with alerts about CORS), something went wrong [here](#cors-settings). In that case it might be good to check the headers of a HTTP request against a file on your bucket:

```bash
curl -X OPTIONS -H 'Origin: http://MY_DOMAIN_NAME' http://BUCKET_URL/static/images/logo-small.png -H "Access-Control-Request-Method: GET"
```

Replace `MY_DOMAIN_NAME` with your instance domain name, `BUCKET_URL` with the URL for your bucket, you can replace the file path with any other valid path on your bucket.

If you see any message, especially a message starting with `<Error><Code>CORSForbidden</Code>`, it didn’t work. Wenn Sie keine Nachricht sehen, funktionierte es.

For an active instance, there may be a handful of files that were created locally during the time between migrating the files to external storage, and restarting the app so it uses the external storage. To ensure that any remaining files are uploaded to external storage after switching over, you can use the following command, which will upload only files that aren't already present in the external storage:

```bash
./bw-dev sync_media_to_s3
```

### Instanz-Konnektor aktualisieren

*Note: You can skip this step if you're running an updated version of BookWyrm; in September 2021 the "self connector" was removed in [PR #1413](https://github.com/bookwyrm-social/bookwyrm/pull/1413)*

In order for the right URL to be used when displaying local book search results, we have to modify the value for the cover images URL base.

Connector data can be accessed through the Django admin interface, located at the url `http://MY_DOMAIN_NAME/admin`. The connector for your own instance is the first record in the database, so you can access the connector with this URL: `https://MY_DOMAIN_NAME/admin/bookwyrm/connector/1/change/`.

The field _Covers url_ is defined by default as `https://MY_DOMAIN_NAME/images`, you have to change it to `https://S3_STORAGE_URL/images`. Then, click the _Save_ button, and voilà!

You will have to update the value for _Covers url_ every time you change the URL for your storage.
