# Variables
COMPOSE_FILE := docker-compose.yml
ENV_FILE := .env
SERVICE_NAME := appserver
IMAGE_NAME := protheus-appserver:2510

# Colors
BLUE := \033[0;34m
GREEN := \033[0;32m
YELLOW := \033[1;33m
RED := \033[0;31m
NC := \033[0m # No Color

.PHONY: help
help: ## Show this help message
	@echo "$(BLUE)═══════════════════════════════════════════════════════$(NC)"
	@echo "$(BLUE)  TOTVS Protheus Docker Management$(NC)"
	@echo "$(BLUE)═══════════════════════════════════════════════════════$(NC)"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "$(GREEN)%-20s$(NC) %s\n", $$1, $$2}'

.PHONY: check-env
check-env: ## Check if .env file exists
	@if [ ! -f $(ENV_FILE) ]; then \
		echo "$(RED)Error: $(ENV_FILE) file not found!$(NC)"; \
		echo "$(YELLOW)Please create .env file from .env.example$(NC)"; \
		exit 1; \
	fi

.PHONY: build
build: check-env ## Build the Docker image
	@echo "$(BLUE)Building Protheus Docker image...$(NC)"
	docker-compose -f $(COMPOSE_FILE) build --no-cache
	@echo "$(GREEN)✓ Build completed successfully!$(NC)"

.PHONY: up
up: check-env ## Start the containers
	@echo "$(BLUE)Starting Protheus containers...$(NC)"
	docker-compose -f $(COMPOSE_FILE) up -d
	@echo "$(GREEN)✓ Containers started successfully!$(NC)"
	@$(MAKE) status

.PHONY: down
down: ## Stop and remove containers
	@echo "$(YELLOW)Stopping Protheus containers...$(NC)"
	docker-compose -f $(COMPOSE_FILE) down
	@echo "$(GREEN)✓ Containers stopped successfully!$(NC)"

.PHONY: restart
restart: ## Restart the containers
	@echo "$(BLUE)Restarting Protheus containers...$(NC)"
	docker-compose -f $(COMPOSE_FILE) restart
	@echo "$(GREEN)✓ Containers restarted successfully!$(NC)"

.PHONY: logs
logs: ## Show container logs
	docker-compose -f $(COMPOSE_FILE) logs -f $(SERVICE_NAME)

.PHONY: shell
shell: ## Access container shell
	@echo "$(BLUE)Accessing container shell...$(NC)"
	docker-compose -f $(COMPOSE_FILE) exec $(SERVICE_NAME) /bin/bash

.PHONY: status
status: ## Show container status
	@echo "$(BLUE)Container Status:$(NC)"
	@docker-compose -f $(COMPOSE_FILE) ps
	@echo ""
	@echo "$(BLUE)Network Information:$(NC)"
	@docker network inspect protheus_net --format='Subnet: {{range .IPAM.Config}}{{.Subnet}}{{end}}' 2>/dev/null || echo "Network not created yet"

.PHONY: clean
clean: down ## Remove containers, volumes and images
	@echo "$(YELLOW)Cleaning up Docker resources...$(NC)"
	@docker volume rm $$(docker volume ls -q | grep protheus) 2>/dev/null || true
	@docker rmi $(IMAGE_NAME) 2>/dev/null || true
	@echo "$(GREEN)✓ Cleanup completed!$(NC)"

.PHONY: rebuild
rebuild: clean build up ## Clean, rebuild and start containers

.PHONY: health
health: ## Check container health
	@echo "$(BLUE)Checking container health...$(NC)"
	@docker inspect --format='{{.State.Health.Status}}' protheus-appserver 2>/dev/null || echo "Container not running"

.PHONY: config
config: check-env ## Validate docker-compose configuration
	@echo "$(BLUE)Validating configuration...$(NC)"
	docker-compose -f $(COMPOSE_FILE) config
	@echo "$(GREEN)✓ Configuration is valid!$(NC)"

.PHONY: info
info: ## Show system information
	@echo "$(BLUE)═══════════════════════════════════════════════════════$(NC)"
	@echo "$(BLUE)  System Information$(NC)"
	@echo "$(BLUE)═══════════════════════════════════════════════════════$(NC)"
	@echo "Docker Version: $$(docker --version)"
	@echo "Docker Compose Version: $$(docker-compose --version)"
	@echo "Image: $(IMAGE_NAME)"
	@echo "Service: $(SERVICE_NAME)"
	@echo ""
	@if [ -f $(ENV_FILE) ]; then \
		echo "$(BLUE)Environment Configuration:$(NC)"; \
		echo "TCP Port: $$(grep TCP_PORT $(ENV_FILE) | cut -d'=' -f2)"; \
		echo "HTTP REST Port: $$(grep HTTPREST_PORT $(ENV_FILE) | cut -d'=' -f2)"; \
		echo "Environment: $$(grep ENV_SERVER $(ENV_FILE) | cut -d'=' -f2)"; \
	fi

.PHONY: backup-logs
backup-logs: ## Backup container logs
	@mkdir -p ./backups
	@docker cp protheus-appserver:/totvs/protheus/bin/log/appserver.log ./backups/appserver_$$(date +%Y%m%d_%H%M%S).log
	@echo "$(GREEN)✓ Logs backed up to ./backups/$(NC)"

.DEFAULT_GOAL := help