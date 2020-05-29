# NiFi

Configuration and deployment for Apache NiFi

## Generating truststore/keystore

Use [nifi-toolkit](https://nifi.apache.org/docs/nifi-docs/html/toolkit-guide.html) to generate:

```sh
./bin/tls-toolkit.sh standalone \
    --certificateAuthorityHostname ${PUBLIC_HOSTNAME} \
    --hostnames ${TRAEFIK_SUBDOMAIN}.${PUBLIC_HOSTNAME} \
    --days 825 \
    --keySize 4096 \
    --trustStorePassword ${TRUSTSTORE_PASSWORD} \
    --keyStorePassword ${KEYSTORE_PASSWORD} \
    --keyPassword ${KEY_PASSWORD}
```

Then, migrate from JKS (Java specific, deprecated format) to PKCS12 (generic, recommended format) using:

```sh
keytool -importkeystore -srckeystore keystore.jks -destkeystore keystore.p12 -deststoretype pkcs12
keytool -importkeystore -srckeystore truststore.jks -destkeystore truststore.p12 -deststoretype pkcs12
```

You can check stores content with:

```sh
keytool -list -v -keystore keystore.p12
keytool -list -v -keystore truststore.p12
```

## Adding truststore/keystore to volume

When deploying at first time, you must copy generated stores to `${SECURITY_VOL_NAME}` volume (at root level). NiFi needs these files at startup.
