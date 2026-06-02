.PHONY: run tail logs clean kill runlive runall count_accepted help update-session compose-emails _check_role

LOCATION ?=
MAXPAGES ?= 10
EMAILS_MODE ?=

_LOCATION_ARG = $(if $(LOCATION),--location "$(LOCATION)",)
_EMAILS_MODE_ARG = $(if $(EMAILS_MODE),--emails-mode,)

_check_role:
	@if [ -z "$(ROLE)" ]; then \
		echo ""; \
		echo "ERROR: ROLE is required."; \
		echo ""; \
		echo "usage:"; \
		echo "  make $(MAKECMDGOALS) ROLE=\"backend engineer\" [LOCATION=\"india\"] [MAXPAGES=10] [EMAILS_MODE=1]"; \
		echo ""; \
		echo "available roles:"; \
		echo "  software engineer, backend engineer, software architect, devops, data engineer, security engineer"; \
		echo ""; \
		echo "available locations:"; \
		echo "  india  (omit for worldwide)"; \
		echo ""; \
		exit 1; \
	fi

run: _check_role
	uv run main.py --role "$(ROLE)" $(_LOCATION_ARG) --maxpages $(MAXPAGES) $(_EMAILS_MODE_ARG)

tail:
	tail -f $$(ls -t logs/*.log | head -1)

logs:
	ls -lt logs/*.log

clean:
	rm -f logs/*.log
	rm -f ./emails/*.log
	rm -f ./emails/composed/*.log
	rm -f ./emails/composed/*.txt

kill:
	tmux kill-session -t wf 2>/dev/null && echo "session 'wf' killed" || echo "no session 'wf' found"
	tmux kill-session -t emailmake 2>/dev/null && echo "session 'emailmake' killed" || echo "no session 'emailmake' found"

update-session:
	mkdir -p config
	@read -p "paste curl command: " curl; \
	uv run config.py update_session "$$curl"

count_accepted:
	uv run emailcount.py

compose-emails:
	@if [ -z "$(FILE)" ]; then \
		echo ""; \
		echo "ERROR: FILE is required."; \
		echo ""; \
		echo "usage:"; \
		echo "  make compose-emails FILE=emails/2026_05_14__14_22.log"; \
		echo ""; \
		exit 1; \
	fi
	@if tmux has-session -t emailmake 2>/dev/null; then \
		read -p "tmux session 'emailmake' already exists. kill it? [y/n]: " ans; \
		if [ "$$ans" = "y" ]; then \
			tmux kill-session -t emailmake; \
			echo "killed existing session"; \
		else \
			echo "skipping, session left running"; \
			exit 0; \
		fi \
	fi; \
	tmux new-session -d -s emailmake -x 220 -y 50; \
	tmux send-keys -t emailmake:0 "uv run email_composer.py $(FILE)" Enter; \
	tmux attach-session -t emailmake

runlive: _check_role
	@if tmux has-session -t wf 2>/dev/null; then \
		read -p "tmux session 'wf' already exists. kill it? [y/n]: " ans; \
		if [ "$$ans" = "y" ]; then \
			tmux kill-session -t wf; \
			echo "killed existing session"; \
		else \
			echo "skipping, session left running"; \
			exit 0; \
		fi \
	fi; \
	tmux new-session -d -s wf -x 220 -y 50; \
	tmux split-window -h -t wf; \
	tmux send-keys -t wf:0.0 "uv run main.py --role \"$(ROLE)\" $(_LOCATION_ARG) --maxpages $(MAXPAGES) $(_EMAILS_MODE_ARG)" Enter; \
	tmux send-keys -t wf:0.1 "sleep 2 && tail -f \$$(ls -t logs/*.log | head -1)" Enter; \
	tmux attach-session -t wf

runall:
	@if tmux has-session -t wf 2>/dev/null; then \
		read -p "tmux session 'wf' already exists. kill it? [y/n]: " ans; \
		if [ "$$ans" = "y" ]; then \
			tmux kill-session -t wf; \
			echo "killed existing session"; \
		else \
			echo "skipping, session left running"; \
			exit 0; \
		fi \
	fi; \
	tmux new-session -d -s wf -x 220 -y 50 -n "se"; \
	tmux send-keys -t wf:0 "uv run main.py --role \"software engineer\" --maxpages $(MAXPAGES) $(_EMAILS_MODE_ARG)" Enter; \
	\
	tmux new-window -t wf -n "be"; \
	tmux send-keys -t wf:1 "uv run main.py --role \"backend engineer\" --maxpages $(MAXPAGES) $(_EMAILS_MODE_ARG)" Enter; \
	\
	tmux new-window -t wf -n "sa"; \
	tmux send-keys -t wf:2 "uv run main.py --role \"software architect\" --maxpages $(MAXPAGES) $(_EMAILS_MODE_ARG)" Enter; \
	\
	tmux new-window -t wf -n "dv"; \
	tmux send-keys -t wf:3 "uv run main.py --role \"devops\" --maxpages $(MAXPAGES) $(_EMAILS_MODE_ARG)" Enter; \
	\
	tmux new-window -t wf -n "de"; \
	tmux send-keys -t wf:4 "uv run main.py --role \"data engineer\" --maxpages $(MAXPAGES) $(_EMAILS_MODE_ARG)" Enter; \
	\
	tmux new-window -t wf -n "sec"; \
	tmux send-keys -t wf:5 "uv run main.py --role \"security engineer\" --maxpages $(MAXPAGES) $(_EMAILS_MODE_ARG)" Enter; \
	\
	tmux new-window -t wf -n "logs"; \
	tmux send-keys -t wf:6 "sleep 3 && tail -f logs/*.log" Enter; \
	\
	tmux select-window -t wf:6; \
	tmux attach-session -t wf

help:
	@echo ""
	@echo "WELLFOUND BOT"
	@echo "============="
	@echo ""
	@echo "RUNNING"
	@echo "  make run ROLE=\"backend engineer\"                        run in current terminal"
	@echo "  make run ROLE=\"devops\" LOCATION=\"india\" MAXPAGES=50     run with location + page limit"
	@echo "  make run ROLE=\"devops\" EMAILS_MODE=1                    run in emails-only mode (no applying)"
	@echo "  make runlive ROLE=\"data engineer\"                       run in tmux with live log tail"
	@echo "  make runlive ROLE=\"devops\" EMAILS_MODE=1                emails-only mode in tmux"
	@echo "  make runall                                              run all 6 roles in tmux windows"
	@echo "  make runall MAXPAGES=100                                 runall with custom page limit"
	@echo "  make runall EMAILS_MODE=1                                runall in emails-only mode"
	@echo ""
	@echo "  required: ROLE"
	@echo "  available roles:  software engineer, backend engineer, software architect,"
	@echo "                    devops, data engineer, security engineer"
	@echo "  available locations: india  (omit for worldwide)"
	@echo ""
	@echo "EMAILS"
	@echo "  make run ROLE=... EMAILS_MODE=1                         extract emails from JDs, skip applying"
	@echo "  make runall EMAILS_MODE=1                               extract emails for all roles"
	@echo "  make compose-emails FILE=emails/2026_05_14__14_22.log   compose cold emails from extracted list"
	@echo "  make count_accepted                                      count accepted applications"
	@echo ""
	@echo "SESSION"
	@echo "  make update-session                                      paste fresh curl to update expired cookies"
	@echo ""
	@echo "LOGGING"
	@echo "  make tail                                                tail the most recent log"
	@echo "  make logs                                                list all logs newest first"
	@echo "  make clean                                               delete all logs and email files"
	@echo ""
	@echo "TMUX"
	@echo "  make kill                                                kill tmux sessions 'wf' and 'emailmake'"
	@echo "  tmux attach -t wf                                        reattach to running bot session"
	@echo "  tmux attach -t emailmake                                 reattach to email composer session"
	@echo "  ctrl+b [0-6]                                            switch tmux windows in runall"
	@echo ""
	@echo "NOTES"
	@echo "  cookies expire every few hours — on 403 run: make update-session"
	@echo "  runall does not require ROLE (runs all 6 roles automatically)"
	@echo "  run and runlive require ROLE — omitting it prints this usage info"
	@echo ""
