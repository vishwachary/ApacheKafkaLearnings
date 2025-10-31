kafka-topics.bat --create --topic publication-render-html --bootstrap-server localhost:9092 --partitions 6 --replication-factor 1
kafka-topics.bat --create --topic publication-render-pdf --bootstrap-server localhost:9092 --partitions 6 --replication-factor 1
kafka-topics.bat --create --topic publication-render-markdown --bootstrap-server localhost:9092 --partitions 6 --replication-factor 1


DELETE  TOPIC


