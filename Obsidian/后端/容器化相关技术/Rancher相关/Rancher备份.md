```
docker run --name busybox-backup-2024-01-09 --volumes-from 819c26791480 -v $PWD:/backup busybox tar pzcvf /backup/rancher-data-backup-2.5.9-2024-01-09.tar.gz /var/lib/rancher

```

```
docker run --volumes-from 819c26791480 -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /var/lib/rancher

```

```
docker run -d \
  --name /rancher \
  --restart=unless-stopped \
  -p 8080:80 -p 8443:443 \
  -v /docker_volume/rancher_home/auditlog:/var/log/auditlog \
  -v /docker_volume/rancher_home/rancher:/var/lib/rancher \
  rancher/rancher:stable
```