# Week 8.2 exercise (`week8.2_Exercise.ipynb`)

This README describes **`week8.2_Exercise.ipynb`**: the course **`DealAgentFramework`** path (RSS deals, ensemble pricing, `memory.json`, optional Gradio). The notebook finds **`week8/`** on disk and `chdir`s there so paths match the rest of the week 8 code.

### If you are not the author (clone / review)

- **You need this full repository** (or at least the course **`week8/`** package: `agents/`, `deal_agent_framework.py`, `price_is_right.py`, etc.). This folder alone is not enough to import `DealAgentFramework`.
- **Python environment:** use the same stack as the LLM engineering course (see the repo root **`requirements.txt`** and the week 8 lessons). Typical packages include `chromadb`, `gradio`, `openai`, `python-dotenv`, `scikit-learn`, and the rest of the course deps.
- **`.env`:** create one (repo root or `week8/`) with **`OPENAI_API_KEY`**. Optional: **`HF_TOKEN`**, Modal auth if you run the full ensemble.
- **Defaults are safe:** leave **`RUN_PIPELINE = False`** and **`LAUNCH_UI = False`** until keys and data are in place; the notebook still runs and prints memory counts (possibly zero).

---

## What the notebook does

1. **Resolve `week8/`**  
   Walks upward from the current working directory until it finds `deal_agent_framework.py`, or `week8/deal_agent_framework.py` (works if your Jupyter cwd is the repo root, the `week8` folder, or this folder). It adds that directory to `sys.path` and **`chdir`s into it** so **`memory.json`** and the **Chroma** folder **`products_vectorstore`** line up with how the course code expects paths.

2. **Environment**  
   Loads `.env` with **`python-dotenv`**.

3. **Inspect `memory.json`**  
   Instantiates **`DealAgentFramework()`**, which reads persisted **`Opportunity`** rows from **`memory.json`** under `week8/`. The next cell prints how many entries exist and shows a short preview of the first deal (listed price, ensemble estimate, discount).

4. **Optional: one planning cycle**  
   With **`RUN_PIPELINE = True`**, calls **`framework.run()`**, which:
   - lazily builds **`PlanningAgent`** (scanner + ensemble + messaging),
   - pulls deals from RSS (Dealnews feeds in `agents/deals.py`),
   - prices them via **`EnsembleAgent`** (frontier RAG + Modal specialist + neural net blend),
   - may **append** a new row to memory **only if** the best deal’s discount is above **`PlanningAgent.DEAL_THRESHOLD`** (default **$50** in code).

   With **`RUN_PIPELINE = False`** (default), nothing network-heavy runs; you only inspect existing memory.

5. **Gradio UI**  
   The **“Price is Right”** app is **`price_is_right.py`** in `week8/`. Run it from a **terminal**, or use the last notebook cell with **`LAUNCH_UI = True`** (imports **`App`** and calls **`App().run()`**; that **blocks the kernel** until you interrupt it). Default is **`LAUNCH_UI = False`**.

---

## How to use it

1. **Place a `.env` file** where you normally keep secrets for this course (often the repo root or `week8/`), with at least **`OPENAI_API_KEY`**. Add **`HF_TOKEN`** if your setup needs it for Hub/embeddings.

2. **Open `week8.2_Exercise.ipynb`** in Cursor, Jupyter, or VS Code. **Kernel:** the same Python environment as the rest of the course **`week8`** stack (Chroma, `openai`, etc.).

3. **Run cells top to bottom** on first use:
   - **Setup** cell: confirms `week8 root:` in the output.
   - **Inspect memory** cell: safe by default; only touches local Chroma + JSON.
   - **Optional pipeline** cell: leave **`RUN_PIPELINE = False`** until Chroma is populated, Modal **`pricer-service`** is deployed, and you are OK with RSS + HTTP traffic. Then set **`RUN_PIPELINE = True`** and re-run that cell.

4. **Launch the Gradio app** (optional):  
   - **Terminal:** `cd` to `week8/` and run `python price_is_right.py`.  
   - **Notebook:** run the setup cell first, then set **`LAUNCH_UI = True`** in the last code cell and run it. That cell **reloads** `deal_agent_framework` and `price_is_right` so you pick up edits without restarting the kernel; if anything still looks stale, use **Kernel → Restart** and run from the top. Use **Kernel → Interrupt** when you want to stop the server.

---

## What you need for a *full* agent run

| Requirement | Why |
|---------------|-----|
| **`OPENAI_API_KEY`** | Scanner and frontier-style pricing use the OpenAI API. |
| **`week8/products_vectorstore`** (Chroma) | **`FrontierAgent`** / RAG path expects the course vector store. |
| **Modal + `pricer-service`** | **`SpecialistAgent`** calls your deployed Modal app by name (`pricer-service` / `Pricer`), as in the lessons. |
| **Network** | RSS feeds and product page fetches for the scanner. |

If you only want to **read** existing **`memory.json`** and print summaries, you can do that without a full Modal deployment, as long as Chroma can open the local DB (Chroma may still initialize on framework construction).

---

## Main knobs

| Name | Role |
|------|------|
| **`RUN_PIPELINE`** | `False`: skip RSS + planner + ensemble run. `True`: run **`DealAgentFramework.run()`** once. |
| **`LAUNCH_UI`** | `False`: skip Gradio in the notebook. `True`: run **`App().run()`** (blocks kernel until interrupt). |
| **`PlanningAgent.DEAL_THRESHOLD`** | In **`week8/agents/planning_agent.py`** (default **50**): minimum **dollar discount** for the planner to treat a deal as worth storing/alerting. |

---

## Files

| File | Purpose |
|------|---------|
| **`week8.2_Exercise.ipynb`** | Described above. |
| **`README.md`** | This guide. |
