# Week 7.2 exercise (`week7.2_Exercise.ipynb`)

This folder holds community exercise notebooks for the LLM engineering course. **`week7.2_Exercise.ipynb`** is a minimal walkthrough for **Week 7**: predict a product’s **price from its text description** using the course **`pricer`** package, a small **open-source** chat model, and (optionally) **QLoRA** fine-tuning.

---

## What the notebook does (in order)

1. **Python path and environment**  
   - Adds the **`week7`** directory (two levels up from the default working directory) to `sys.path` so `import pricer` works.  
   - Loads `.env` via `python-dotenv` and, if `HF_TOKEN` is set, logs into Hugging Face Hub.

2. **Data**  
   - Loads **`ed-donner/items_lite`** or **`ed-donner/items_full`** from the Hub (controlled by `LITE_MODE`).  
   - Splits: train / validation / test as provided by the dataset.  
   - Fills missing `summary` with `title`.

3. **Tokenizer and prompts**  
   - Loads **`TinyLlama/TinyLlama-1.1B-Chat-v1.0`** tokenizer.  
   - For **train + validation**: builds `prompt` / `completion` with truncated summaries; **prices are rounded** in the completion (`do_round=True`).  
   - For **test**: same truncation; **exact prices** in the completion (`do_round=False`) so evaluation matches the real test labels.  
   - Truncation uses at most **`SUMMARY_TOKEN_CAP = 110` summary tokens** (same idea as the main Week 7 materials).

4. **Baseline inference (no fine-tuning)**  
   - Loads **`TinyLlama/TinyLlama-1.1B-Chat-v1.0`** as a causal LM: **4-bit NF4** on **CUDA**, else **float16 on MPS** or **float32 on CPU**.  
   - For each test item, builds the **test prompt** (prompt only, ending at “Price is $”), runs **greedy generation** (`max_new_tokens=16`), decodes the new tokens, and extracts the first number with a small regex as the predicted price.  
   - Runs **`pricer.evaluator.evaluate`**: MAE, MSE, R², scatter plot, and error-trend plot.  
   - Evaluation uses **`EVAL_SIZE`** items (capped by test set size). **`workers`** is **5 on CUDA** and **1 otherwise** (single-threaded on CPU/MPS avoids unstable parallel inference with some backends).

5. **Optional QLoRA training (off by default)**  
   - If **`RUN_QLORA = True`** and **CUDA is available**, trains a **LoRA adapter** on the **first `MAX_SAMPLES` training rows** only.  
   - Training text is **`prompt + completion`** concatenated into a single `text` field for `SFTTrainer`.  
   - Base model is loaded in **4-bit**; LoRA targets **`q_proj`, `k_proj`, `v_proj`, `o_proj`** with **`r=8`**, **`lora_alpha=16`**, **`lora_dropout=0.05`**.  
   - Checkpoints / adapter weights are written to  
     **`week7/community_contributions/sammyloto/qlora_tinyllama_pricer`** (relative to the repo), via `OUTPUT_DIR` in the notebook.

6. **Optional evaluation of the adapter**  
   - Set **`ADAPTER_PATH`** to the folder that contains the saved PEFT adapter (e.g. the same `OUTPUT_DIR` after training).  
   - Loads **base model + PeftModel**, then runs the **same** `price_from_model` path as the baseline and calls **`evaluate`** again with the same `EVAL_SIZE` and worker rules.

---

## What you need

- **Run the notebook with working directory set to `week7`** (or adjust `week7_root` in the first code cell so it points at the `week7` folder that contains `pricer/`).  
- Python deps used by the course `week7` stack: **`transformers`**, **`torch`**, **`datasets`**, **`python-dotenv`**, **`huggingface_hub`**, **`tqdm`**, **`scikit-learn`**, **`pandas`**, **`plotly`**, plus **`pydantic`** for `Item`.  
- For QLoRA: **`peft`**, **`bitsandbytes`**, **`trl`**, **`accelerate`** (CUDA strongly recommended for training).  
- Optional: **`HF_TOKEN`** in `.env` for Hub access if your environment requires it.

---

## Main knobs (in the notebook)

| Name | Role |
|------|------|
| `LITE_MODE` | `True` → `items_lite`; `False` → `items_full`. |
| `SUMMARY_TOKEN_CAP` | Max summary tokens before truncation when building prompts. |
| `EVAL_SIZE` | How many test points the evaluator runs on. |
| `RUN_QLORA` | Set `True` on CUDA to run training (otherwise skipped). |
| `MAX_SAMPLES` | Training rows used when QLoRA runs. |
| `ADAPTER_PATH` | Folder with saved LoRA weights; `None` skips adapter evaluation. |

---

## Files in this folder

| File | Purpose |
|------|--------|
| `week7.2_Exercise.ipynb` | Week 7.2 TinyLlama baseline + optional QLoRA + optional adapter eval (this README describes it). |
| `week7_Exercise.ipynb` | Earlier Week 7 variant using **Qwen2.5-1.5B-Instruct** and a separate adapter output path. |

After a local QLoRA run, you may also see **`qlora_tinyllama_pricer/`** here (adapter + tokenizer files), depending on what you saved.
