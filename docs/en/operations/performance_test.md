---
toc_priority: 54
toc_title: Testing Hardware
---

# How To Test Your Hardware With ClickHouse {#how-to-test-your-hardware-with-clickhouse}

With this instruction you can run basic ClickHouse performance test on any server without installation of ClickHouse packages.

1.  Go to “commits” page: https://github.com/ClickHouse/ClickHouse/commits/master

2.  Click on the first green check mark or red cross with green “ClickHouse Build Check” and click on the “Details” link near “ClickHouse Build Check”.

3.  Copy the link to “clickhouse” binary for amd64 or aarch64.

4.  ssh to the server and download it with wget:

<!-- -->

      # For amd64:
      wget https://clickhouse-builds.s3.yandex.net/0/00ba767f5d2a929394ea3be193b1f79074a1c4bc/1578163263_binary/clickhouse
      # For aarch64:
      wget https://clickhouse-builds.s3.yandex.net/0/00ba767f5d2a929394ea3be193b1f79074a1c4bc/1578161264_binary/clickhouse
      # Then do:
      chmod a+x clickhouse

1.  Download configs:

<!-- -->

      wget https://raw.githubusercontent.com/ClickHouse/ClickHouse/master/programs/server/config.xml
      wget https://raw.githubusercontent.com/ClickHouse/ClickHouse/master/programs/server/users.xml
      mkdir config.d
      wget https://raw.githubusercontent.com/ClickHouse/ClickHouse/master/programs/server/config.d/path.xml -O config.d/path.xml
      wget https://raw.githubusercontent.com/ClickHouse/ClickHouse/master/programs/server/config.d/log_to_console.xml -O config.d/log_to_console.xml

1.  Download benchmark files:

<!-- -->

      wget https://raw.githubusercontent.com/ClickHouse/ClickHouse/master/benchmark/clickhouse/benchmark-new.sh
      chmod a+x benchmark-new.sh
      wget https://raw.githubusercontent.com/ClickHouse/ClickHouse/master/benchmark/clickhouse/queries.sql

1.  Download test data according to the [Yandex.Metrica dataset](../getting_started/example_datasets/metrica.md) instruction (“hits” table containing 100 million rows).

<!-- -->

      wget https://clickhouse-datasets.s3.yandex.net/hits/partitions/hits_100m_obfuscated_v1.tar.xz
      tar xvf hits_100m_obfuscated_v1.tar.xz -C .
      mv hits_100m_obfuscated_v1/* .

1.  Run the server:

<!-- -->

      ./clickhouse server

1.  Check the data: ssh to the server in another terminal

<!-- -->

      ./clickhouse client --query "SELECT count() FROM hits_100m_obfuscated"
      100000000

1.  Edit the benchmark-new.sh, change “clickhouse-client” to “./clickhouse client” and add “–max\_memory\_usage 100000000000” parameter.

<!-- -->

      mcedit benchmark-new.sh

1.  Run the benchmark:

<!-- -->

      ./benchmark-new.sh hits_100m_obfuscated

1.  Send the numbers and the info about your hardware configuration to clickhouse-feedback@yandex-team.com

All the results are published here: https://clickhouse.tech/benchmark\_hardware.html
