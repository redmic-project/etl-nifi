# NiFi

Configuration and deployment for Apache NiFi

## Generating truststore/keystore

Use [nifi-toolkit](https://nifi.apache.org/docs/nifi-docs/html/toolkit-guide.html) (included in NiFi container image) to generate:

```sh
./bin/tls-toolkit.sh standalone \
    --hostnames ${TRAEFIK_SUBDOMAIN} \
    --certificateAuthorityHostname ${TRAEFIK_SUBDOMAIN} \
    --nifiDnSuffix ",${LDAP_USER_SEARCH_BASE}" \
    --subjectAlternativeNames "localhost,0.0.0.0,${TRAEFIK_SUBDOMAIN}.${PUBLIC_HOSTNAME},${TRAEFIK_SUBDOMAIN}" \
    --days 825 \
    --keySize 4096 \
    --trustStorePassword ${TRUSTSTORE_PASSWORD} \
    --keyStorePassword ${KEYSTORE_PASSWORD} \
    --keyPassword ${KEY_PASSWORD} \
    --outputDirectory ${NIFI_HOME}/security
```

If you want to generate truststore/keystore for NiFi Registry, you also have to execute:

```sh
./bin/tls-toolkit.sh standalone \
    --hostnames ${NIFI_REGISTRY_TRAEFIK_SUBDOMAIN} \
    --certificateAuthorityHostname ${NIFI_REGISTRY_TRAEFIK_SUBDOMAIN} \
    --nifiDnSuffix ",${LDAP_USER_SEARCH_BASE}" \
    --subjectAlternativeNames "localhost,0.0.0.0,${NIFI_REGISTRY_TRAEFIK_SUBDOMAIN}.${PUBLIC_HOSTNAME},${NIFI_REGISTRY_TRAEFIK_SUBDOMAIN}" \
    --days 825 \
    --keySize 4096 \
    --trustStorePassword ${TRUSTSTORE_PASSWORD} \
    --keyStorePassword ${KEYSTORE_PASSWORD} \
    --keyPassword ${KEY_PASSWORD} \
    --outputDirectory ${NIFI_HOME}/security
```

You can check stores content with:

```sh
keytool -list -v -keystore keystore.jks
keytool -list -v -keystore truststore.jks
```

## Adding truststore/keystore to volume

When deploying at first time, you must copy generated stores to `${SECURITY_VOL_NAME}` volume (at root level). NiFi needs these files at startup.

## Adding external service certificate

In processes like `invokeHttp`, if you wish to access to an external service through HTTPS where a certificate is mandatory, you must add it to NiFi's truststore, using these commands:

```sh
echo -n | openssl s_client -connect external_url:443 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > /tmp/external_name.crt

keytool -import -alias external_name -file /tmp/external_name.crt -keystore truststore.jks

rm tmp/external_name.crt
```
