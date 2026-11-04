# Beyond Consistency: Extension-Based Argumentation for LLMs

Code, prompts, data samples and results for the paper
**"Beyond Consistency: Extension-Based Argumentation Framework for Reasoning in Large Language Models"**
(submitted to *Expert Systems with Applications*).

Given a claim, the pipeline asks an LLM for facts and for arguments for and against the claim,
filters the arguments by quality, audits their support and attack relations, builds a bipolar
argumentation framework in which opposing arguments may attack each other, and reads the verdict
from its grounded, preferred and stable extensions with a decision rule
(credulous, skeptical, majority or No-objection).

## Repository structure

```
notebooks/   pipeline and baseline code (Jupyter, originally run on Kaggle)
prompts/     all prompts, exactly as sent to the models (= Appendix A of the paper)
data/        sampled claims used in the experiments
results/     per-claim outputs and summary results
```

## Setup

```bash
pip install -r requirements.txt
```

The models are called through OpenAI-compatible APIs. Set your keys as environment variables
(the notebooks read them from Kaggle secrets; replace `UserSecretsClient().get_secret(...)`
with `os.environ[...]` when running elsewhere):

| Model | Provider | Model id | Key |
|---|---|---|---|
| DeepSeek-V3 | DeepSeek API | `deepseek-chat` | `DEEPSEEK_API_KEY` |
| Llama 3.3 70B | OpenRouter | `meta-llama/llama-3.3-70b-instruct` | `OPENROUTER_API_KEY` |
| GPT-OSS 120B | OpenRouter | `openai/gpt-oss-120b` | `OPENROUTER_API_KEY` |
| Gemma 4 31B | OpenRouter | `google/gemma-4-31b-it` | `OPENROUTER_API_KEY` |

**Never commit API keys.**

## Configuration

All thresholds are fixed and identical for every model and dataset (`BAF_CONFIG` in the notebook):
up to 5 candidate arguments per side, top 4 kept; relevance ≥ 0.60, grounding ≥ 0.55,
clarity ≥ 0.60, quality ≥ 0.58; relation audit: target match ≥ 0.65, grounding ≥ 0.55,
relation score ≥ 0.68; at most 3 conflict pairs and 2 claim attacks;
generation temperature 0.15, audit temperature 0.

## Data

| Dataset | Source | Sample |
|---|---|---|
| StrategyClaim | ArgLLM repository (`Datasets/StrategyQA/Experiment`), 500-claim split with annotators' facts | loaded directly from https://github.com/CLArg-group/argumentative-llms |
| COM2SENSE | training split, 250 true / 250 false | `data/samples/com2sense_sample_500.csv` |
| αNLI | development split, 250 plausible / 250 implausible hypotheses | drawn with seed 42 (see paper, Section 5.1) |
| CosmosQA | validation split, 250 correct / 250 wrong answers | drawn with seed 42 (see paper, Section 5.1) |

All samples are balanced and drawn with a fixed random seed (42). Please respect the licence
of each original dataset.

## Results


- `results/summary/confusion_matrices.csv`: confusion matrices (TN, FP, FN, TP) collected for the
  reported settings.
- `results/per_claim/`: per-claim outputs (arguments, relations, frameworks, extensions and
  verdicts) for DeepSeek-V3 on COM2SENSE.

Per-claim outputs are provided for the runs that were preserved; for the other runs, only the
aggregate results reported in the paper are available. All runs can be reproduced with the code
and the samples above.

## Baselines

- **ArgLLM** (0.5 Base and Est. Base), depth-one configuration, DF-QuAD from the authors'
  code (https://github.com/CLArg-group/argumentative-llms). Three prompt versions appear in
  Table 1 of the paper, and each notebook says which one it uses:
  - *original prompts* (rows marked * and StrategyClaim);
  - *re-implemented prompts* (rows marked f: GPT-OSS and Llama on CosmosQA): strengths given
    between 0 and 1, unreadable answers set to 0.5;
  - *earlier adaptation* (Gemma and GPT-OSS on COM2SENSE, DeepSeek on αNLI and CosmosQA): the
    model writes its own facts and must give both arguments, on an earlier sample.
- **Direct questioning** and **net support**: in the notebook.

## Citation

Citation details will be added after publication.

## Licence

Code: MIT (see `LICENSE`). Datasets remain under their original licences.
