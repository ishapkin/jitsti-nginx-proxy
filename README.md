# jitsti-nginx-proxy
docker-compose.yml example for using docker-jitsi-meet in your existing nginx-proxy/nginx-proxy + nginx-proxy/docker-letsencrypt-nginx-proxy-companion environment

 - Install docker-jitsi-meet https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-docker
 - Replace their docker-compose.yml for this docker-compose.yml
 
 ```yml
 version: '3'

services:
  # Frontend
  web:
    image: jitsi/web:latest
    restart: ${RESTART_POLICY}
    expose:
      - 80
      - 443
    volumes:
      - ${CONFIG}/web:/config:Z
      - ${CONFIG}/web/letsencrypt:/etc/letsencrypt:Z
      - ${CONFIG}/transcripts:/usr/share/jitsi-meet/transcripts:Z
    environment:
      - ENABLE_AUTH
      - ENABLE_GUESTS
      - ENABLE_LETSENCRYPT=0
      - ENABLE_HTTP_REDIRECT
      - ENABLE_TRANSCRIPTIONS
      - DISABLE_HTTPS
      - JICOFO_AUTH_USER
      - LETSENCRYPT_DOMAIN
      - PUBLIC_URL
      - XMPP_DOMAIN
      - XMPP_AUTH_DOMAIN
      - XMPP_BOSH_URL_BASE
      - XMPP_GUEST_DOMAIN
      - XMPP_MUC_DOMAIN
      - XMPP_RECORDER_DOMAIN
      - ETHERPAD_URL_BASE
      - ETHERPAD_PUBLIC_URL
      - TZ
      - JIBRI_BREWERY_MUC
      - JIBRI_PENDING_TIMEOUT
      - JIBRI_XMPP_USER
      - JIBRI_XMPP_PASSWORD
      - JIBRI_RECORDER_USER
      - JIBRI_RECORDER_PASSWORD
      - ENABLE_RECORDING
      - VIRTUAL_HOST=meet.domain.org
      - LETSENCRYPT_HOST=meet.domain.org
      - LETSENCRYPT_EMAIL=mail@example.com
    networks:
      meet.jitsi:
        aliases:
          - ${XMPP_DOMAIN}
      default:

  # XMPP server
  prosody:
    image: jitsi/prosody:latest
    restart: ${RESTART_POLICY}
    expose:
      - '5222'
      - '5347'
      - '5280'
    volumes:
      - ${CONFIG}/prosody/config:/config:Z
      - ${CONFIG}/prosody/prosody-plugins-custom:/prosody-plugins-custom:Z
    environment:
      - AUTH_TYPE
      - ENABLE_AUTH
      - ENABLE_GUESTS
      - ENABLE_LOBBY
      - GLOBAL_MODULES
      - GLOBAL_CONFIG
      - LDAP_URL
      - LDAP_BASE
      - LDAP_BINDDN
      - LDAP_BINDPW
      - LDAP_FILTER
      - LDAP_AUTH_METHOD
      - LDAP_VERSION
      - LDAP_USE_TLS
      - LDAP_TLS_CIPHERS
      - LDAP_TLS_CHECK_PEER
      - LDAP_TLS_CACERT_FILE
      - LDAP_TLS_CACERT_DIR
      - LDAP_START_TLS
      - XMPP_DOMAIN
      - XMPP_AUTH_DOMAIN
      - XMPP_GUEST_DOMAIN
      - XMPP_MUC_DOMAIN
      - XMPP_INTERNAL_MUC_DOMAIN
      - XMPP_MODULES
      - XMPP_MUC_MODULES
      - XMPP_INTERNAL_MUC_MODULES
      - XMPP_RECORDER_DOMAIN
      - JICOFO_COMPONENT_SECRET
      - JICOFO_AUTH_USER
      - JICOFO_AUTH_PASSWORD
      - JVB_AUTH_USER
      - JVB_AUTH_PASSWORD
      - JIGASI_XMPP_USER
      - JIGASI_XMPP_PASSWORD
      - JIBRI_XMPP_USER
      - JIBRI_XMPP_PASSWORD
      - JIBRI_RECORDER_USER
      - JIBRI_RECORDER_PASSWORD
      - JWT_APP_ID
      - JWT_APP_SECRET
      - JWT_ACCEPTED_ISSUERS
      - JWT_ACCEPTED_AUDIENCES
      - JWT_ASAP_KEYSERVER
      - JWT_ALLOW_EMPTY
      - JWT_AUTH_TYPE
      - JWT_TOKEN_AUTH_MODULE
      - LOG_LEVEL
      - TZ
    networks:
      meet.jitsi:
        aliases:
          - ${XMPP_SERVER}

  # Focus component
  jicofo:
    image: jitsi/jicofo:latest
    restart: ${RESTART_POLICY}
    volumes:
      - ${CONFIG}/jicofo:/config:Z
    environment:
      - AUTH_TYPE
      - ENABLE_AUTH
      - XMPP_DOMAIN
      - XMPP_AUTH_DOMAIN
      - XMPP_INTERNAL_MUC_DOMAIN
      - XMPP_SERVER
      - JICOFO_COMPONENT_SECRET
      - JICOFO_AUTH_USER
      - JICOFO_AUTH_PASSWORD
      - JICOFO_RESERVATION_REST_BASE_URL
      - JVB_BREWERY_MUC
      - JIGASI_BREWERY_MUC
      - JIGASI_SIP_URI
      - JIBRI_BREWERY_MUC
      - JIBRI_PENDING_TIMEOUT
      - TZ
    depends_on:
      - prosody
    networks:
      meet.jitsi:

  # Video bridge
  jvb:
    image: jitsi/jvb:latest
    restart: ${RESTART_POLICY}
    ports:
      - '${JVB_PORT}:${JVB_PORT}/udp'
      - '${JVB_TCP_PORT}:${JVB_TCP_PORT}'
    volumes:
      - ${CONFIG}/jvb:/config:Z
    environment:
      - DOCKER_HOST_ADDRESS
      - XMPP_AUTH_DOMAIN
      - XMPP_INTERNAL_MUC_DOMAIN
      - XMPP_SERVER
      - JVB_AUTH_USER
      - JVB_AUTH_PASSWORD
      - JVB_BREWERY_MUC
      - JVB_PORT
      - JVB_TCP_HARVESTER_DISABLED
      - JVB_TCP_PORT
      - JVB_STUN_SERVERS
      - JVB_ENABLE_APIS
      - TZ
    depends_on:
      - prosody
    networks:
      meet.jitsi:

# Custom network so all services can communicate using a FQDN
networks:
  meet.jitsi:
  default:
    external:
      name: nginx-proxy

 ```
 
 - meet.domain.org and mail@example.com must be replaced with your mail and domain
 
