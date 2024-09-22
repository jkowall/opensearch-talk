# OpenSearchCon Talk 2024

Here are the source files for the talk if you are interested. You'll need Docker but that's about it.

## Housekeeping

Setup the following hosts on your local machine

127.0.0.1 loadgenerator ngnix frontend dashboards featureflag Grafana jaeger

## URLs for testing
URLs
APP : http://frontend:8080/
Loadgen : http://loadgenerator:8089/
Jaeger UI : http://localhost:16686
OpenSearch Dash : http://dashboards:5601/app/observability-traces#/traces
Grafana to show the trace query to Jaeger : http://localhost:3000/grafana

## More Details

Based on [https://opentelemetry.io/docs/demo/architecture/](https://opentelemetry.io/docs/demo/architecture/) thanks to [Yang DB](https://github.com/YANG-DB) for the work. I have modified it as per below:

```mermaid
Observability Stack
graph LR
subgraph tdf[Telemetry Data Flow]
  subgraph subgraph_padding [ ]
    style subgraph_padding fill:none,stroke:none;
    subgraph od[OpenTelemetry Demo]
      ms(Microservice)
    end
    ms -->|"OTLP<br/>gRPC"| oc-grpc
    ms -->|"OTLP<br/>HTTP POST"| oc-http

    subgraph oc[OTel Collector]
      style oc fill:#97aef3,color:black;
      oc-grpc[/"OTLP Receiver"/]
      oc-http[/"OTLP Receiver"/]
      oc-proc(Processors)
      oc-prom[/"OTLP HTTP Exporter"/]
      oc-otlp[/"OTLP Exporter"/]
      oc-grpc --> oc-proc
      oc-http --> oc-proc
      oc-proc --> oc-prom
      oc-proc --> oc-otlp
    end

    oc-prom -->|gRPC| pr-sc
    oc-otlp -->|gRPC| ot-col 

    subgraph pr[Prometheus]
      style pr fill:#e75128,color:black;
      pr-sc[/"Prometheus OTLP Write Receiver"/]
      pr-tsdb[(Prometheus TSDB)]
      pr-http[/"Prometheus HTTP"/]
      pr-sc --> pr-tsdb
      pr-tsdb --> pr-http
    end

    pr-b{{"Browser<br/>Prometheus UI"}}
    pr-http ----> pr-b

    subgraph ja[Jaeger]
      style ja fill:#60d0e4,color:black;
      ja-http[/"Jaeger HTTP"/]

      subgraph main_database[Main Database]
        opensearch1[/"OpenSearch 1"/]
        opensearch2[/"OpenSearch 2"/]
      end

      subgraph data_prepper[Data Prepper]
        data_prepper_node[/"Data Prepper"/]
      end

      subgraph jaeger_collector[Jaeger Collector]
        jaeger_collector_node[/"Jaeger Collector"/] 
      end

      ot-col --> data_prepper_node
      data_prepper_node --> opensearch1
      data_prepper_node --> opensearch2
      opensearch1 --> ja-http
      opensearch2 --> ja-http
      jaeger_collector_node --> ja-http
    end

    subgraph gr[Grafana]
      style gr fill:#f8b91e,color:black;
      gr-srv["Grafana Server"]
      gr-http[/"Grafana HTTP"/]
      gr-srv --> gr-http
      gr-srv --> opensearch1
      gr-srv --> opensearch2
    end

    pr-http --> gr-srv
    ja-http --> gr-srv

    ja-b{{"Browser<br/>Jaeger UI"}}
    ja-http ----> ja-b

    gr-b{{"Browser<br/>Grafana UI"}}
    gr-http --> gr-b

    osd-b{{"Browser<br/>OpenSearch Dashboards UI"}}
    opensearch1 --> osd-b
    opensearch2 --> osd-b
  end
end
```

