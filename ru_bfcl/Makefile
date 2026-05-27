QWEN4B = qwen3-4b-instruct
RUADAPT = ruadapt-qwen3-4b
QWEN35 = qwen35moe35b
QWEN235B = Qwen3-235B-A22B-Instruct-2507
MODELS = $(QWEN4B) $(RUADAPT) $(QWEN35) $(QWEN235B)

TEST_CATEGORIES = non_live,live,multi_turn

prepare:
ifndef BASE_URL
	@echo "Please export BASE_URL first"
	return 1
endif
ifdef BASE_URL
	@echo "Preparing bfcl-eval package..."
	python3.10 -m venv .venv
	. .venv/bin/activate && pip install bfcl-eval==2025.7.9 soundfile \
		&& cp $$(python -c "import bfcl_eval; print(bfcl_eval.__path__[0])")/.env.example .env
	echo "" >> .env
	echo "MSU_API_KEY=" >> .env
	@echo "Configuring models..."
	sed 's|base_url.*|base_url="$(BASE_URL)",|' .venv/lib/python3.10/site-packages/bfcl_eval/model_handler/api_inference/qwen.py --in-place
	sed 's/api_key=.*$$/api_key=os.getenv("MSU_API_KEY"),/' .venv/lib/python3.10/site-packages/bfcl_eval/model_handler/api_inference/qwen.py --in-place
	cp model_config.py .venv/lib/python3.10/site-packages/bfcl_eval/constants/model_config.py
	@echo "Updating dataset..."
	-git clone https://huggingface.co/datasets/AvitoTech/ru_bfcl
	rm -rf .venv/lib/python3.10/site-packages/bfcl_eval/data/*
	mv ru_bfcl/* .venv/lib/python3.10/site-packages/bfcl_eval/data
	rm -rf ru_bfcl
	@echo "Done. Please add API key to .env"
endif

$(MODELS):
	export BFCL_PROJECT_ROOT=$$(pwd) \
		&& . .venv/bin/activate  \
		&& bfcl generate --model $@ --test-category $(TEST_CATEGORIES)
	export BFCL_PROJECT_ROOT=$$(pwd) \
		&& . .venv/bin/activate  \
		&& bfcl evaluate --model $@

eval_all: $(MODELS)

clean:
	rm -rf .venv

.PHONY: prepare eval_all clean $(MODELS)
