# root credentials that will be posteriorly and God knows how
# exported to a Java project
include .env
export

MACHINE           ?= 4kgbbad
DRIVER            ?= mysql
DRIVER_VERSION    ?= 8.3
APP               ?= app2

VOLUME       := $(DRIVER)-$(MACHINE)-$(DRIVER_VERSION)-volume
NETWORK      := $(MACHINE)-net
DB_CONTAINER      := $(DRIVER)-$(MACHINE)-1
INIT_TEMPLATE     := initdb.sql
OUT_DIR 			:= out
GENERATED_SQL     := $(OUT_DIR)/$(APP)-initdb.sql

## first and OG not meant to be re-generated
.PHONY: mysql-bootstrap
mysql-bootstrap:
	@echo "Creating container: $(DB_CONTAINER)"
	@docker run -d \
		--name $(DB_CONTAINER) \
		--restart unless-stopped \
		--network $(NETWORK) \
		-p 3306:3306 \
		-v $(VOLUME):/var/lib/mysql \
		-e MYSQL_ROOT_PASSWORD=$(DB_ROOT_PASSWORD) \
		-e MYSQL_DATABASE=$(DB_DATABASE) \
		-e MYSQL_USER=$(DB_USER) \
		-e MYSQL_PASSWORD=$(DB_PASSWORD) \
		$(DRIVER):$(DRIVER_VERSION) > /dev/null
	@echo " $(DB_CONTAINER) ready and running."

# === Find next clone suffix
CONTAINER_SUFFIX := $(shell \
  INDEX=2; \
  while docker ps -a --format '{{.Names}}' | grep -q "^$(DB_CONTAINER)-$$INDEX$$"; do \
    INDEX=$$(($$INDEX + 1)); \
  done; \
  echo $$INDEX \
)

DB_CONTAINER_OUT := $(DRIVER)-$(MACHINE)-$(CONTAINER_SUFFIX)

# === Generate SQL script from template
$(GENERATED_SQL): $(INIT_TEMPLATE)
	@mkdir -p $(OUT_DIR)
	@echo "️  Generating SQL for app: $(APP)"
	@sed 's/__APP__/$(APP)/g' $(INIT_TEMPLATE) > $(GENERATED_SQL)
	@echo " SQL written to $(GENERATED_SQL)"
	
OUT_RUN_CMD := $(OUT_DIR)/$(APP)-docker-run.sh

.PHONY: mysql-app-bootstrap
mysql-app-bootstrap: $(GENERATED_SQL)
	@echo "⚙️  Running init script in $(DB_CONTAINER)..."
	@docker exec -i $(DB_CONTAINER) \
	  mysql -uroot -p$(DB_ROOT_PASSWORD) < $(GENERATED_SQL)

	@echo "🔍 Verifying that $(APP)_db and user $(APP) were created..."
	@docker exec -i $(DB_CONTAINER) \
	  mysql -uroot -p$(DB_ROOT_PASSWORD) \
	  -e "SHOW DATABASES; SELECT user, host FROM mysql.user WHERE user='$(APP)';"

	@echo "#!/bin/bash" > $(OUT_RUN_CMD)
	@echo "docker run -d \\" >> $(OUT_RUN_CMD)
	@echo "  --name $(DB_CONTAINER_OUT) \\" >> $(OUT_RUN_CMD)
	@echo "  --restart unless-stopped \\" >> $(OUT_RUN_CMD)
	@echo "  --network $(NETWORK) \\" >> $(OUT_RUN_CMD)
	@echo "  -v $(VOLUME):/var/lib/mysql \\" >> $(OUT_RUN_CMD)
	@echo "  $(DRIVER):$(DRIVER_VERSION)" >> $(OUT_RUN_CMD)
	@chmod +x $(OUT_RUN_CMD)
	@echo "Docker run command written to $(OUT_RUN_CMD)"

.PHONY: clean-out
clean-out:
	@echo "🧹 Cleaning out generated files..."
	@rm -rf $(OUT_DIR)/*
	@echo "✅ All cleaned up."

.PHONY: list-containers
list-containers:
	@echo "📦 Containers matching $(DRIVER)-$(MACHINE)*:"
	@docker ps -a --filter "name=$(DRIVER)-$(MACHINE)" --format "  - {{.Names}} ({{.Status}})"

.PHONY: inspect-db
inspect-db:
	@echo "🔍 Inspecting $(APP)_db and user $(APP)..."
	@docker exec -i $(DB_CONTAINER) \
	  mysql -uroot -p$(DB_ROOT_PASSWORD) \
	  -e "SHOW GRANTS FOR '$(APP)'@'%'; USE $(APP)_db; SHOW TABLES;"


