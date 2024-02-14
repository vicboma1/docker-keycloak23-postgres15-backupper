# docker-keycloak23-postgres15-backupper

```
version: '3.9'
services:
  postgres:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_PASSWORD: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_DB: keycloak
    volumes:
      - type: bind
        source: ./storage/postgres
        target: /var/lib/postgresql/data
    networks:
      - net-keycloak

  keycloak:
    depends_on:
      - postgres
    image: quay.io/keycloak/keycloak:23.0
    restart: always
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: keycloak
    ports:
      - 9090:8080
    command:
      - start-dev
    networks:
      - net-keycloak

    # https://github.com/prodrigestivill/docker-postgres-backup-local
    # https://pkg.go.dev/github.com/robfig/cron#hdr-Predefined_schedules
  pgbackups:
    depends_on:
      - postgres
    container_name: local_pgbackups
    image: prodrigestivill/postgres-backup-local
    restart: always
    #user: postgres:postgres # Optional: see below
    volumes:
      - type: bind
        source: ./backups
        target: /backups
    links:
      - postgres
    depends_on:
      - postgres
    environment:
      TZ: Europe/Paris
      POSTGRES_CLUSTER: true
      POSTGRES_HOST: postgres
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: keycloak
      #  - POSTGRES_PASSWORD_FILE=/run/secrets/db_password <-- alternative for POSTGRES_PASSWORD (to use with docker secrets)
      POSTGRES_EXTRA_OPTS: -Z6 --schema=public --blobs
      SCHEDULE: "@every 0h10m00s"
      BACKUP_KEEP_DAYS: 7
      BACKUP_KEEP_WEEKS: 4
      BACKUP_KEEP_MONTHS: 6
      HEALTHCHECK_PORT: 8080
    networks:
     - net-keycloak

networks:
  net-keycloak:
```
