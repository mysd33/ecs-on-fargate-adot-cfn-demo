FROM quay.io/keycloak/keycloak:latest AS builder

WORKDIR /opt/keycloak

# for demonstration purposes only, please make sure to use proper certificates in production instead
RUN keytool -genkeypair -storepass password -storetype PKCS12 -keyalg RSA -keysize 2048 -dname "CN=server" -alias server -ext "SAN:c=DNS:localhost,IP:127.0.0.1" -keystore conf/server.keystore

# Add Amazon Web Services JDBC Driver to the providers directory before building Keycloak
# https://www.keycloak.org/server/db#preparing-keycloak-for-amazon-aurora-postgresql
ADD --chmod=0666 https://github.com/awslabs/aws-advanced-jdbc-wrapper/releases/download/4.0.1/aws-advanced-jdbc-wrapper-4.0.1.jar /opt/keycloak/providers/aws-advanced-jdbc-wrapper.jar

# Build-time options (needed for 'start --optimized')
ENV KC_HEALTH_ENABLED=true
ENV KC_METRICS_ENABLED=true
ENV KC_DB=postgres
ENV KC_DB_DRIVER=software.amazon.jdbc.Driver

# Build
RUN /opt/keycloak/bin/kc.sh build

# Runtime image
FROM quay.io/keycloak/keycloak:latest
# Copy build artifacts (including providers built above) from the builder stage
COPY --from=builder /opt/keycloak/ /opt/keycloak/

ENTRYPOINT ["/opt/keycloak/bin/kc.sh"]