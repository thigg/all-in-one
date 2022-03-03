## Reverse Proxy Config

Basically, you need to specify the port that the apache container shall use and modify the startup command a bit.

All examples below will use port `11000` as example apache port. Also it is supposed that the reverse proxy runs on the same server like AIO, hence `localhost` is used and not an internal ip-address to point to the AIO instance. Modify both to your needings.

### Caddy reverse proxy config example

Add this to your Caddyfile:

```
https://<your-nc-domain>:443 {
    header Strict-Transport-Security max-age=31536000;
    reverse_proxy localhost:11000
}
```

Of course you need to modify `<your-nc-domain>` to the domain on which you want to run Nextcloud.

### Startup command

```
# For x64 CPUs:
sudo docker run -it \
--name nextcloud-aio-mastercontainer \
--restart always \
-p 8080:8080 \
-e APACHE_PORT=11000 \
--volume nextcloud_aio_mastercontainer:/mnt/docker-aio-config \
--volume /var/run/docker.sock:/var/run/docker.sock:ro \
nextcloud/all-in-one:latest
```

<details>

<summary>Command for arm64 CPUs like the Raspberry Pi 4</summary>

```
# For arm64 CPUs:
sudo docker run -it \
--name nextcloud-aio-mastercontainer \
--restart always \
-p 8080:8080 \
-e APACHE_PORT=11000 \
--volume nextcloud_aio_mastercontainer:/mnt/docker-aio-config \
--volume /var/run/docker.sock:/var/run/docker.sock:ro \
nextcloud/all-in-one:latest-arm64
```

</details>

After doing so, you should be able to access the AIO Interface via `https://internal.ip.of.this.server:8080`. Enter your domain that you've entered in the reverse proxy config and you should be done. Please do not forget to open port `3478/TCP` and `3478/UDP` for the Talk container!

### Optional

If you want to also access your AIO interface publicly with a valid certificate, you can add e.g. the following config to your Caddyfile:

```
https://<your-nc-domain>:8443 {
    reverse_proxy https://localhost:8080 {
        transport http {
            tls_insecure_skip_verify
        }
    }
}
```

Of course you also need to modify `<your-nc-domain>` to the domain that you want to use. Afterwards should the AIO interface be accessible via `https://<your-nc-domain>:8443`.