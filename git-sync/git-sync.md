- https://hub.docker.com/r/openweb/git-sync/

```yaml
# docker-compose.yml

version: '3'

volumes:
  sources:
    driver: local

services:
  nginx:
    container_name: nginx
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - sources:/usr/share/nginx/html
    depends_on:
      - git-sync
    restart: always
  git-sync:
    container_name: git-sync
    image: openweb/git-sync:0.0.1
    environment:
      GIT_SYNC_REPO: "https://github.com/socar-hardy/socar-data-airflow-v2-beta"
      GIT_SYNC_DEST: "/git"
      GIT_SYNC_BRANCH: "master"
      GIT_SYNC_REV: "FETCH_HEAD"
      GIT_SYNC_WAIT: "10"
    volumes:
      - sources:/git
    restart: always
```

```bash
$ docker-compose up -d
```

```
Creating volume "git-sync_sources" with local driver
Creating git-sync ... done
Creating nginx    ... done
```



```bash
$ git-sync docker volume inspect git-sync_sources
[
    {
        "CreatedAt": "2020-07-06T07:13:18Z",
        "Driver": "local",
        "Labels": {
            "com.docker.compose.project": "git-sync",
            "com.docker.compose.version": "1.25.0-rc2",
            "com.docker.compose.volume": "sources"
        },
        "Mountpoint": "/var/lib/docker/volumes/git-sync_sources/_data",
        "Name": "git-sync_sources",
        "Options": null,
        "Scope": "local"
    }
]
```



```bash
$ git-sync sudo ls /var/lib/docker/volumes/git-sync_sources/_data
README.md  dags
```

