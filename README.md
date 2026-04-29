# rulyotano.infra
My simple local infrastructure

## Restoring volumes

Before running the volume.restore action. We need to copy the volume backup that we want to restore, into the directory `~/volumes/`. It needs to be already unzipped.


For example:

```
 tar -xf drupal-sites-0000000000.tar.zip
 scp drupal-sites-0000000000.tar root@vpn2.rulyotano.com:./volumes/drupal-sites.tar
```
