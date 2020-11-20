---
title: "How to Deploy Quarkus on Platform.sh with Elasticsearch"
sidebarTitle: "Elasticsearch"
weight: -110
layout: single
toc: false
description: |
    Configure a Quarkus application with Elasticsearch.
---

Quarkus provides two ways of accessing Elasticsearch: via the lower level `RestClient` or via the `RestHighLevelClient` we will call them the low level and the high level clients.

To activate Elasticsearch and then have it accessed by the Quarkus application already in Platform.sh, it is necessary to modify two files. [There is also instruction in case it is necessary to move an application from scratch](_index.md).

* The first file is the services, where it will include Elasticsearch as a service.

  ```yaml
  searchelastic:
    type: elasticsearch:7.2
    disk: 1024
  ```

* The second and last file is to grant access to the service to the application; otherwise, it won't access it.

```yaml
name: app
type: "java:11"
disk: 1024
hooks:
    build: ./mvnw package -DskipTests -Dquarkus.package.uber-jar=true
relationships:
    search: "searchelastic:elasticsearch"
web:
    commands:
        start: java -jar $JAVA_OPTS target/file.jar
```

To simplify the application file, there is [Shell variables](https://docs.platform.sh/development/variables.html#shell-variables) int the  `.environment` file. This way,  it does not need to change the application file, only the environment file.

```properties
export ES_HOST=`echo $PLATFORM_RELATIONSHIPS|base64 -d|json_pp|jq -r ".search[0].host"`
export ES_PORT=`echo $PLATFORM_RELATIONSHIPS|base64 -d|json_pp|jq -r ".search[0].port"`
export QUARKUS_HIBERNATE_SEARCH_ELASTICSEARCH_HOSTS=${ES_HOST}:${ES_PORT}
export QUARKUS_HTTP_PORT=$PORT
export JAVA_OPTS="-Xmx$(jq .info.limits.memory /run/config.json)m -XX:+ExitOnOutOfMemoryError"
```
{{< note title="Tip">}}
Environment variables names are following the conversion rules of [Eclipse MicroProfile](https://github.com/eclipse/microprofile-config/blob/master/spec/src/main/asciidoc/configsources.asciidoc#default-configsources)
{{< /note >}}
Commit that code and push. The application is ready and connected to a Elasticsearch instance.