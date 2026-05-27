MSU_MODEL = $(shell envsubst < msu_model)

QWEN4B = qwen3-4b-instruct
RUADAPT = ruadapt-qwen3-4b
QWEN35 = qwen35moe35b
QWEN235B = Qwen3-235B-A22B-Instruct-2507
MODELS = $(QWEN4B) $(RUADAPT) $(QWEN35) $(QWEN235B)

prepare:
ifndef BASE_URL
	@echo "Please export BASE_URL first"
	return 1
endif
ifdef BASE_URL
	@echo "Preparing tau repository..."
	-git clone https://github.com/sierra-research/tau2-bench
	cd tau2-bench && uv sync
	cp .env.example tau2-bench/.env
	@echo "Configuring models..."
	sed 's|^  }$$|  },\n$(MSU_MODEL)|' tau2-bench/.venv/lib/python3.12/site-packages/litellm/llms/openai_like/providers.json --in-place
	sed '123s/response/return 1\n    response/' tau2-bench/src/tau2/utils/llm_utils.py --in-place
	@echo "Done. Please add API key to tau2-bench/.env"
endif

$(MODELS):
	for domain in airline retail telecom banking_knowledge ; do       \
		cd tau2-bench && . .venv/bin/activate                     \
		&& tau2 run --agent-llm msu/$@ --user-llm msu/$(QWEN235B) \
			--max-retries 0 --domain $$domain ;               \
	done

eval_all: $(MODELS)

clean:
	cd tau2-bench && make clean

.PHONY: prepare eval_all clean $(MODELS)
