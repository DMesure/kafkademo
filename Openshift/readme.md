# Setup overview for Kafka, using Strimzi and monitoring with Prometheus and Grafana

For a more indepth explanation visit the written overview on:
https://district09.atlassian.net/wiki/spaces/SF/pages/edit-v2/3064955044

Or my information source:
https://snourian.com/kafka-kubernetes-strimzi-part-1-creating-deploying-strimzi-kafka/

### This overview will simply show what to apply in what order to recreate the exact project.
Directories where files are used will be mentioned above each section.

--> /install/cluster-operator

In every \*RoleBinding\*.yaml file, change the "dmesure-strimzi-dv" namespace to your namespace.
Apply the whole folder.

    oc apply /cluster-operator

--> /examples/kafka

    oc apply -f customKafkaDefinition.yaml

--> /examples/kafka

    oc apply -f KafkaTopic.yaml

### Testing the Kafka Cluster
Using 2 predefined images provided by strimzi we can quickly test the pub-sub pattern on our Kafka Cluster.

Open 2 terminal windows and log in to your openshift cluster.
Producer: 

    oc run kafka-producer -ti --image=strimzi/kafka:0.20.0-rc1-kafka-2.6.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic my-topic

Consumer: 

    oc run kafka-consumer -ti --image=strimzi/kafka:0.20.0-rc1-kafka-2.6.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning

The producer opens an interactive terminal where you can type a message and send it by pressing enter. You should see it appearing in the consumer terminal.

--> /prometheus/prometheus.yaml

Change every reference of "namespace: dmesure-strimzi-dv" to your own namespace.

    oc apply -f prometheus.yaml

--> /examples/metrics/prometheus-install

Look for every reference of the dmesure-strimzi-dv namespace and change it to your own.

    oc apply -f strimzi-pod-monitor.yaml

    oc apply -f  prometheus-rules.yaml

    oc apply -f prometheus.yaml

-->  /examples/metriscs/grafana-install

    oc apply -f grafana.yaml

    oc expose svc grafana

Go to Networking --> Routes in the OpenShift webview and click on the newly created Grafana url.

It opens a dashboard, the default username and password are: admin and admin. You will be prompted to change this immediately.

On the dashboard add a datasource in the settings.
In the url put : http://prometheus-operated:9090
Leave the rest empty and press Save & Test.

Under dashboards go to manage and press import.
We imported the strimzi-kafka.json, strimzi-kafka-exporter.json, strimzi-operators.json and strimzi-zookeeper.json which can be found under /examples/metrics/grafana-dashboards.

Don’t forget to select Prometheus as a datasource.

That's it, you should now be able to see data appearing on the newly created dashboards!