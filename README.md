# elk-snapshot
ELK - Snapshot

Fuente: https://github.com/elastic/helm-charts/blob/v7.11.1/elasticsearch/examples/minikube/values.yaml

```sh
$ kubectl create namespace elasticsnapshot
$ kubectl config set-context --current --namespace=elasticsnapshot

# Crear el PVC
kubectl apply -f pvc_snapshot.yml -n elasticsnapshot

$ helm install elasticsearch elastic/elasticsearch -f elasticsearch-master/values.yaml

$ helm install kibana elastic/kibana -f elasticsearch-kibana/values.yaml

```


PUT /_ingest/pipeline/test
  {
    "description" : "Pipeline Log A",
    "processors" : [
      {
        "grok" : {
          "field" : "message",
          "patterns" : [
            """\A%{WORD:level} %{TIMESTAMP_ISO8601:timestamp} \[%{WORD:thread}\] %{JAVACLASS:class} - %{WORD:type}\|%{NUMBER:duration}\|%{DATA:action}\|%{WORD:status}\|%{USERNAME:username}""",
            """\A%{WORD:level} %{TIMESTAMP_ISO8601:timestamp} \[%{WORD:thread}\] %{JAVACLASS:class} - %{WORD:type}\|%{NUMBER:status}\|%{USERNAME:username}\|%{IPV4:ip}""",
            """\A(?m)%{WORD:level} %{TIMESTAMP_ISO8601:timestamp} \[%{WORD:thread}\] %{JAVACLASS:class} - %{DATA:description}\|%{GREEDYDATA:stacktrace}""",
            """\A%{WORD:level} %{TIMESTAMP_ISO8601:timestamp} \[%{WORD:thread}\] %{JAVACLASS:class} - %{GREEDYDATA:description}"""
          ],
          "tag" : "test_logs"
        }
      },
      {
        "date" : {
          "field" : "timestamp",
          "formats" : [
            "yyyy-MM-dd HH:mm:ss"
          ],
          "timezone" : "Europe/Madrid",
          "tag" : "test_timestamp"
        }
      },{
        "remove": {
          "field": ["input"]
        }
      },
      {
        "convert": {
          "field": "duration",
          "type": "integer",
          "if": "ctx.level == 'PERFORMANCE'"
        }
      }
    ],
    "on_failure" : [
      {
        "set" : {
          "field" : "_index",
          "value" : "failed-{{ _index }}"
        }
      },
      {
        "set" : {
          "field" : "error",
          "value" : "{{ _ingest.on_failure_message }}"
        }
      }
    ]
  }


$ helm install elastic-filebeat elastic/filebeat -f elasticsearch-filebeat/values.yaml
