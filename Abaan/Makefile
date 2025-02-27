# Makefile for Django project

SHELL := /bin/bash

.ONESHELL:

# Define variables
.DEFAULT_GOAL := help


PORT = 8000
IP = localhost

LOCAL_SETTINGS = config.settings
SETTINGS = $(LOCAL_SETTINGS)

# Define the virtual environment name
VENV_NAME = .venv

PIPENVIRON = pipenv
# Automatically get the project and app directories
PROJECT_DIR := $(shell dirname $(CURDIR))
APP_DIR := $(CURDIR)

PROJECT_NAME := $(shell basename "$(APP_DIR)")

# Ensure virtual environment is active before running any commands
PIPENV_SHELL = pipenv shell
PIPENV_RUN = pipenv run

PYTHON ?= python3
PIP ?= pip3
# Define manage.py command as $(MANAGE) for convenience
MANAGE = $(PIPENV_RUN) $(PYTHON) manage.py

SCRIPTS_DIR := config/scripts

export PYTHON_PATH := $(shell which $(PYTHON) 2>/dev/null)
export PIP_PATH := $(shell which $(PIP) 2>/dev/null)
export PIPENV := $(shell which $(PIPENVIRON) 2> /dev/null)

# Default target when `make` is called without arguments
all:
	@echo "Running Server..."
	@gnome-terminal --tab --title="Server" --command="bash -c 'make runserver SETTINGS=$(SETTINGS); exec bash'" > /dev/null 2>&1
	@echo "Server running successfully."

dir:
	@echo "PROJECT_DIR: $(PROJECT_DIR)"
	@echo "APP_DIR: $(APP_DIR)"
	@echo "PROJECT_NAME: $(PROJECT_NAME)"  
	
# Activate virtual environment
venv:
	@gnome-terminal --tab --title="Virtual environment" --command="bash -c '$(PIPENV_SHELL); exec bash'" > /dev/null 2>&1
	@echo "Virtual environment running successfully."
	

collectstatic:  ## Collect static files
	$(MANAGE) collectstatic --noinput

# Run development server on localhost
runserver:
	$(MANAGE) runserver --settings=$(SETTINGS) $(IP):$(PORT)

# Create database tables and apply migrations
migrate:
	$(MANAGE) makemigrations
	$(MANAGE) migrate

# Create a new app (replace `myapp` with your desired app name)
createapp:
	@read -p "Enter the name of the app: " app_name; \
	$(MANAGE) startapp $$app_name; \
	mv $$app_name $(APP_DIR)/apps/
	@echo "$$app_name app created successfully."
	$(PYTHON) $(SCRIPTS_DIR)/add_to_installed_apps.py $$app_name
	$(PYTHON) $(SCRIPTS_DIR)/edit_apps.py $(APP_DIR)/apps/$$app_name $$app_name; \


# Run tests
test:
	$(MANAGE) test

# Clean up unused files
clean:
	find . -name "*.pyc" -delete
	find . -name "__pycache__" -delete

check-python-pip:
	@echo "Checking for $(PYTHON) and $(PIP)..."

	@if [ -z "$(PYTHON_PATH)" ]; then \
		echo "$(PYTHON) is not installed. Please install $(PYTHON) first."; \
	else \
		echo "Found $(PYTHON) at $(PYTHON_PATH):"; \
		$(PYTHON_PATH) --version; \

	fi

	@if [ -z "$(PIP_PATH)" ]; then \
		echo "$(PIP) is not installed. Please install $(PIP) first."; \
	else \
		echo "Found $(PIP) at $(PIP_PATH):"; \
		$(PIP_PATH) --version; \
	fi

install-pipenv:
	@if [ -z "$$PIP_PATH" ]; then \
		echo "$(PIP) is not installed. Please install $(PIP) first."; \
		exit 1; \
	fi
	$$PIP_PATH install pipenv

install-dependencies: 
	export PIPENV_VENV_IN_PROJECT=1
	pipenv install

setup: check-python-pip install-pipenv install-dependencies


uninstall-dependencies:
	@read -p "Are you sure you want to uninstall all dependencies? (Y/n) " confirm; \
	if [ "$$confirm" = "y" ] || [ "$$confirm" = "Y" ]; then \
		$$PIP_PATH uninstall pipenv
	else \
		echo "Uninstall cancelled."; \
	fi

COLOR_RED = \033[0;31m
COLOR_GREEN = \033[0;32m
COLOR_YELLOW = \033[0;33m
COLOR_RESET = \033[0m

define confirm_prompt
$(COLOR_YELLOW)Renaming the project name will result in the deletion of the current Virtual Environment.$(COLOR_RESET)
$(COLOR_YELLOW)This is necessary to ensure that the new project name is selected correctly in the virtual environment.$(COLOR_RESET)
$(COLOR_YELLOW)A new Virtual Environment will be installed with the updated project name.$(COLOR_RESET)
$(COLOR_GREEN)Are you sure you want to continue? (Y/n) : $(COLOR_RESET)
endef
export confirm_prompt

define change_shell
$(COLOR_YELLOW)Kindly change to the renamed directory$(COLOR_RESET)
endef
export change_shell

rename:
	@read -p "Enter the new project name: " new_name; \
	if [ -z "$$new_name" ]; then \
		echo "Project name cannot be empty."; \
	else
		printf "$$confirm_prompt"; \
		read -r confirm; \
		if [ "$$confirm" = "y" ] || [ "$$confirm" = "Y" ]; then \
			rm -rf .venv
			sed -i 's/PROJECT_NAME := $(PROJECT_NAME)/PROJECT_NAME := $$new_name/g' Makefile; \
			# Rename the project directory
			mv $(PROJECT_DIR)/$(PROJECT_NAME) $(PROJECT_DIR)/$$new_name; \
			rm -rf $(APP_DIR)
			echo "Project successfully renamed to $$new_name..."; \
			cd $(PROJECT_DIR)/$$new_name; \
			tput setaf 3; \
			echo "Warning: You are still in the old directory, kindly change your directory to $(PROJECT_DIR)/$$new_name"; \
			tput sgr0; \
			gnome-terminal --working-directory=$(PROJECT_DIR)/$$new_name -- bash -c "make setup && make all"; \

		else \
			echo "Project renaming cancelled."; \
		fi
	fi

# Help target to display available commands
help:
	@echo "Available commands:"
	@echo "  all            - Run project"
	@echo "  venv           - Activate virtual environment"
	@echo "  runserver      - Run development server on localhost."
	@echo "  migrate        - Apply migrations and create database tables."
	@echo "  createapp      - Create a new app (replace 'myapp' with your desired app name)."
	@echo "  test           - Run tests."
	@echo "  clean          - Clean up unused files."
	@echo "  collectstatic  - Collect static files."
