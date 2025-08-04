MACHINE           ?= 4kgbbad
DRIVER            ?= postgres
DRIVER_VERSION    ?= 17.5
APP               ?= cauth

POSTGRES_PREFIX   := pg
VOLUME            := $(POSTGRES_PREFIX)-$(MACHINE)-$(DRIVER_VERSION)-volume
DB_CONTAINER      := $(POSTGRES_PREFIX)-$(MACHINE)-$(DRIVER_VERSION)
NETWORK           := $(MACHINE)-net

DB                ?= postgres
DB_PASSWORD       ?= your_root_password

OUT_DIR           := out
INIT_TEMPLATE     := initdb.sql
GENERATED_SQL     := $(OUT_DIR)/$(APP)-init.sql

# ==== CONFIGURABLE PARAMETERS ====
# Default values (can be overridden when running make)
MACHINE           ?= 4kgbbad
DRIVER            ?= postgres
DRIVER_VERSION    ?= 17.5
POSTGRES_PREFIX   := pg
APP               ?= cauth
APP_NAME          ?= central-authentication

# Derived identifiers
VOLUME       := $(POSTGRES_PREFIX)-$(MACHINE)-$(DRIVER_VERSION)-volume
DB_CONTAINER      := $(POSTGRES_PREFIX)-$(MACHINE)-$(DRIVER_VERSION)
APP_CONTAINER     := $(APP)-1
NETWORK      := $(MACHINE)-net

OUT_DIR 			:= out
INIT_SQL          ?= ./$(OUT_DIR)/$(APP)-init.sql

db-bootstrap: 
	docker volume create $(VOLUME)
	docker network create $(NETWORK) || true
	docker run -d \
		--name $(DB_CONTAINER) \
		--restart unless-stopped \
		--health-cmd="pg_isready -U $(DB)" \
		--health-interval=10s \
		--health-timeout=5s \
		--health-retries=5 \
		-p 5432:5432 \
		-e POSTGRES_DB=$(DB) \
		-e POSTGRES_USER=$(DB) \
		-e POSTGRES_PASSWORD=$(DB_PASSWORD) \
		-v $(VOLUME):/var/lib/postgresql/data \
		--network $(NETWORK) \
		$(DRIVER):$(DRIVER_VERSION)

# ==== APP PROJECT STRUCTURE GENERATOR ====
generate-app-structure:
	@mkdir -p $(APP)
	@echo "📁 Creating app directory: $(APP)"
	@echo "📄 Writing template init.sql..."
	@echo 'DROP SCHEMA IF EXISTS __APP__ CASCADE;' > $(T_SQL)
	@echo 'DROP ROLE IF EXISTS __APP__;' >> $(T_SQL)
	@echo 'CREATE ROLE __APP__ LOGIN PASSWORD '\''__APP___pass'\'';' >> $(T_SQL)
	@echo 'CREATE SCHEMA __APP__ AUTHORIZATION __APP__;' >> $(T_SQL)
	@echo 'REVOKE ALL ON SCHEMA __APP__ FROM PUBLIC;' >> $(T_SQL)
	@echo 'GRANT USAGE ON SCHEMA __APP__ TO __APP__;' >> $(T_SQL)
	@echo 'GRANT CREATE ON SCHEMA __APP__ TO __APP__;' >> $(T_SQL)
	@echo 'GRANT USAGE ON SCHEMA __APP__ TO postgres;' >> $(T_SQL)
	@echo 'GRANT CREATE ON SCHEMA __APP__ TO postgres;' >> $(T_SQL)
	@echo 'ALTER DEFAULT PRIVILEGES IN SCHEMA __APP__ GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO __APP__;' >> $(T_SQL)
	@echo 'ALTER DEFAULT PRIVILEGES IN SCHEMA __APP__ GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO postgres;' >> $(T_SQL)

	@echo "📜 Generating final init.sql..."
	@sed 's/__APP__/$(APP_PG)/g' $(T_SQL) > $(INIT_SQL)

	@echo "📦 Generating Makefile with psql init command..."
	@echo 'run-init:' > $(APP)/Makefile
	@echo '	docker run --rm --network $(NETWORK) \' >> $(APP)/Makefile
	@echo '	-e PGPASSWORD=$(DB_PASSWORD) \' >> $(APP)/Makefile
	@echo '	-v $$(pwd)/$(APP)-init.sql:/init.sql:ro \' >> $(APP)/Makefile
	@echo '	$(DRIVER):$(DRIVER_VERSION) psql -h $(DB_CONTAINER) -U $(DB) -d $(DB) -f /init.sql' >> $(APP)/Makefile

	@echo "📘 Generating docker-compose.yml..."
	@echo 'services:' > $(APP)/docker-compose.yml
	@echo '  $(APP):' >> $(APP)/docker-compose.yml
	@echo '    image: $(APP_NAME):1.0.0' >> $(APP)/docker-compose.yml
	@echo '    container_name: $(APP_CONTAINER)' >> $(APP)/docker-compose.yml
	@echo '    restart: unless-stopped' >> $(APP)/docker-compose.yml
	@echo '    networks:' >> $(APP)/docker-compose.yml
	@echo '      - $(NETWORK)' >> $(APP)/docker-compose.yml
	@echo '    environment:' >> $(APP)/docker-compose.yml
	@echo '      SPRING_DATASOURCE_URL: jdbc:postgresql://$(DB_CONTAINER):5432/postgres' >> $(APP)/docker-compose.yml
	@echo '      SPRING_DATASOURCE_USERNAME: $(APP_PG)' >> $(APP)/docker-compose.yml
	@echo '      SPRING_DATASOURCE_PASSWORD: $(APP)_pass' >> $(APP)/docker-compose.yml
	@echo '      SPRING_JPA_PROPERTIES_HIBERNATE_DEFAULT_SCHEMA: $(APP_PG)' >> $(APP)/docker-compose.yml
	@echo '    depends_on:' >> $(APP)/docker-compose.yml
	@echo '      - $(POSTGRES_PREFIX)' >> $(APP)/docker-compose.yml
	@echo '' >> $(APP)/docker-compose.yml
	@echo 'networks:' >> $(APP)/docker-compose.yml
	@echo '  $(NETWORK):' >> $(APP)/docker-compose.yml
	@echo '    external: true' >> $(APP)/docker-compose.yml

	@echo "📄 Generating application.yml..."
	@APP_TITLE="$$\(echo $(APP_NAME) | sed -E 's/-/ /g' | sed -E 's/\b(.)/\U\1/g') Service"; \
	echo "spring:" > $(APP)/application.yml; \
	echo "  application:" >> $(APP)/application.yml; \
	echo "    name: $$APP_TITLE" >> $(APP)/application.yml; \
	echo "  profiles:" >> $(APP)/application.yml; \
	echo "    active: dev" >> $(APP)/application.yml; \
	echo "  config:" >> $(APP)/application.yml; \
	echo "    import: optional:file:.env[.properties]" >> $(APP)/application.yml; \
	echo "  datasource:" >> $(APP)/application.yml; \
	echo "    url: jdbc:postgresql://$(DB_CONTAINER):5432/postgres" >> $(APP)/application.yml; \
	echo "    username: $(APP_PG)" >> $(APP)/application.yml; \
	echo "    password: $(APP)_pass" >> $(APP)/application.yml; \
	echo "  jpa:" >> $(APP)/application.yml; \
	echo "    hibernate:" >> $(APP)/application.yml; \
	echo "      ddl-auto: update" >> $(APP)/application.yml; \
	echo "    show-sql: true" >> $(APP)/application.yml