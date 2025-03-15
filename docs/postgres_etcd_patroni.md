# Установка и настройка etcd

## Предварительная настройка системы

### Сетевые настройки
```bash
# /etc/hosts
192.168.0.71    etcd1   etcd01
192.168.0.72    etcd2   etcd02
192.168.0.73    etcd3   etcd03
192.168.0.74    node1.postgres  node01
192.168.0.75    node2.postgres  node02
```

### Установка необходимых утилит
```
dnf install -y wget tar 
```

### Отключение firewall (Необходимо будет добавить правильные настройки)
```
systemctl stop firewalld
systemctl disable firewalld
```

## Установка и настройка etcd
### Шаг 1: скачиваем etcd
Оффициальный git etcd: https://github.com/etcd-io/etcd/

```
cd /tmp
wget https://github.com/etcd-io/etcd/releases/download/v3.5.19/etcd-v3.5.19-linux-amd64.tar.gz
tar -xzvf etcd-v3.5.19-linux-amd64.tar.gz
mv etcd-v3.5.19-linux-amd64/etcd* /usr/local/bin/
```

### Шаг 2: создаем пользователей и директории
1. Создаем пользователя:
```
sudo groupadd --system etcd
sudo useradd -s /sbin/nologin --system -g etcd etcd
```

2. Создаем  директорию и предоставляем права
```
mkdir -p /var/lib/etcd /etc/default/etcd/.tls
chown -R etcd:etcd /var/lib/etcd /etc/default/etcd
```

3. Гененрация самоподписанных сертификатов

Содержимое generate_etcd_certs.sh:
```bash
#!/bin/bash

# Директория для сертификатов:
CERT_DIR="/etc/default/etcd/.tls"
mkdir -p ${CERT_DIR}
cd ${CERT_DIR}

# Создание CA-сертификата:
openssl genrsa -out ca.key 4096
openssl req -x509 -new -key ca.key -days 10000 -out ca.crt -subj "/C=RU/ST=Moscow Region/L=Moscow/O=MyOrg/OU=MyUnit/CN=myorg.com"

# Функция для генерации сертификатов для нод:
generate_cert() {
    NODE_NAME=$1
    NODE_IP=$2

    cat <<EOF > ${CERT_DIR}/${NODE_NAME}.san.conf
[ req ]
default_bits       = 4096
distinguished_name = req_distinguished_name
req_extensions     = req_ext
[ req_distinguished_name ]
countryName                 = RU
stateOrProvinceName         = Moscow Region
localityName                = Moscow
organizationName            = MyOrg
commonName                  = ${NODE_NAME}
[ req_ext ]
subjectAltName = @alt_names
[ alt_names ]
DNS.1   = ${NODE_NAME}
IP.1    = ${NODE_IP}
IP.2    = 127.0.0.1
EOF

openssl genrsa -out ${NODE_NAME}.key 4096
openssl req -config ${NODE_NAME}.san.conf -new -key ${NODE_NAME}.key -out ${NODE_NAME}.csr -subj "/C=RU/ST=Moscow Region/L=Moscow/O=MyOrg/CN=${NODE_NAME}"
    openssl x509 -extfile ${NODE_NAME}.san.conf -extensions req_ext -req -in ${NODE_NAME}.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out ${NODE_NAME}.crt -days 10000

    rm -f ${NODE_NAME}.san.conf ${NODE_NAME}.csr
}

# Список нод etcd и их IP-адресов:
ETCD_NODES=("etcd01" "etcd02" "etcd03")
ETCD_IPS=("192.168.0.71" "192.168.0.72" "192.168.0.73")

# Список нод Patroni и их IP-адресов:
PATRONI_NODES=("node01" "node02")
PATRONI_IPS=("192.168.0.74" "192.168.0.75")


# Генерация сертификатов для нод etcd:
for i in "${!ETCD_NODES[@]}"; do
generate_cert "${ETCD_NODES[$i]}" "${ETCD_IPS[$i]}"
done

# Генерация сертификатов для нод Patroni:
for i in "${!PATRONI_NODES[@]}"; do
generate_cert "${PATRONI_NODES[$i]}" "${PATRONI_IPS[$i]}"
done


chown -R etcd:etcd ${CERT_DIR}
chmod 600 ${CERT_DIR}/*.key
chmod 644 ${CERT_DIR}/*.crt

echo "Сертификаты успешно сгенерированы и сохранены в ${CERT_DIR}"

```

Запуск скрипта:
```
chmod +x generate_etcd_certs.sh
./generate_etcd_certs.sh
```

Чтобы узлы etcd взаимодействовали между собой, копируем ca.crt и node[01 или 03] на остальные узлы.

Теперь меняем права на всех узлах etcd:
```
chown -R etcd:etcd /etc/default/etcd/.tls
chmod -R 744 /etc/default/etcd/.tls
chmod 600 /etc/default/etcd/.tls/*.key
```

### Шаг 4: определяем, как должен работать etcd (параметры)
На каждой ноде создаем конфиг, который минимально отличается от ноды к ноде (подсветили комментариями):
```bash
# /etc/etcd/etcd.conf.yml
name: etcd01 # Изменить на других нодах
data-dir: /var/lib/etcd/default
listen-peer-urls: https://0.0.0.0:2380
listen-client-urls: https://0.0.0.0:2379
advertise-client-urls: https://etcd01:2379 # Изменить на других нодах
initial-advertise-peer-urls: https://etcd01:2380 # Изменить на других нодах
initial-cluster-token: etcd_scope
initial-cluster: etcd01=https://etcd01:2380,etcd02=https://etcd02:2380,etcd03=https://etcd03:2380
initial-cluster-state: new
election-timeout: 5000
heartbeat-interval: 500
 
client-transport-security:
  cert-file: /etc/default/etcd/.tls/etcd01.crt # Изменить на других нодах
  key-file: /etc/default/etcd/.tls/etcd01.key
  client-cert-auth: true
  trusted-ca-file: /etc/default/etcd/.tls/ca.crt
 
peer-transport-security:
  cert-file: /etc/default/etcd/.tls/etcd01.crt # Изменить на других нодах
  key-file: /etc/default/etcd/.tls/etcd01.key
  client-cert-auth: true
  trusted-ca-file: /etc/default/etcd/.tls/ca.crt
```

### Шаг 5: для запуска etcd cоздаем Systemd-Unit-файл

Содержимое /etc/systemd/system/etcd.service:
```bash
[Unit]
Description=etcd key-value store
Documentation=https://etcd.io/docs/
Wants=network-online.target
After=network-online.target

[Service]
User=etcd
Type=notify
ExecStart=/usr/local/bin/etcd --config-file=/etc/etcd/etcd.conf.yml
Restart=always
RestartSec=5
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target

```

### Шаг 6: запуск службы etcd
```
systemctl daemon-reload
systemctl enable etcd
systemctl start etcd
```

### Шаг 7: настраиваем alias для etcdctl
```bash
echo 'alias ectl="etcdctl --cacert=/etc/default/etcd/.tls/ca.crt --cert=/etc/default/etcd/.tls/$(hostname).crt --key=/etc/default/etcd/.tls/$(hostname).key --endpoints=https://etcd01:2379,https://etcd02:2379,https://etcd03:2379"' >> ~/.bashrc
source ~/.bashrc
```

Благодаря этому мы получаем статус кластера etcd, не прописывая каждый раз сертификаты.

*Примечание. Учтите, что имя crt и key нод должно быть аналогично hostname. Иначе команда выше не сработает и придется все настраивать вручную.*

Проверка кластера:
```bash
ectl endpoint status -w table

# Ожидаемый рещультат
+---------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|      ENDPOINT       |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+---------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| https://etcd01:2379 | a31313c7cc1b7f7d |  3.5.19 |   20 kB |     false |      false |         4 |         12 |                 12 |        |
| https://etcd02:2379 | c64bbcb095108b13 |  3.5.19 |   20 kB |     false |      false |         4 |         12 |                 12 |        |
| https://etcd03:2379 | 8d1d6ac9f0021f86 |  3.5.19 |   20 kB |      true |      false |         4 |         12 |                 12 |        |
+---------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+

```

После того как кластер запустился, на всех узлах редактируем конфиг `/etc/etcd/etcd.conf.yml` и устанавливаем параметр:
```
***
initial-cluster-state: existing
***
```
Перезапускаем службу:
```
systemctl restart etcd
```