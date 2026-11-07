# Language-model data for Part III (notebooks 16-19)

Real outputs of a hosted language model, cached once so that notebooks 16-19 run offline, fast
and identically for everyone. [generate_llm_data.ipynb](generate_llm_data.ipynb) produced every
file here; you do not need to run it.

## Provenance

- Chat model: `gpt-4.1-mini`, model version `2025-04-14` (the API reported
  `gpt-4.1-mini-2025-04-14` in every response), Azure OpenAI API version `2025-04-01-preview`.
- Embedding model: `text-embedding-3-large`.
- Generated on 2026-11-07 with random seed 20260707 (see `generation_log.json`).
- Why this model: multiple-choice prediction sets need a probability for every option. In a test
  call before generation, `gpt-4.1-mini` returned the 20 most likely first tokens with their
  log-probabilities, while a GPT-5-family reasoning deployment on the same account (`gpt-5.4`)
  returned the log-probability of its chosen token only, with no alternatives.
- Prompts: every prompt is written out in the generator notebook (sections 1-3), next to the code
  that sent it.
- Volume: 19,772 chat requests (about 4.5 million prompt and 0.45 million completion tokens)
  and 140 embedding requests (about 1.1 million tokens), roughly USD 3 at list prices.
- Azure's content filter refused a handful of prompts; the affected questions were dropped
  entirely (7 MMLU history questions, 1 HotpotQA question), so every variant of every remaining
  question is complete.

A hosted model is not bit-for-bit reproducible, even at temperature 0: notebook 16 measures this
on the `rerun` columns. Re-running the generation without the request cache would therefore give
slightly different numbers; these files are the reference for the course.

## Sources and licences

| Source | Used for | Licence |
|---|---|---|
| MMLU (Hendrycks et al., 2021), 28 non-medical subjects, test split | multiple-choice questions | MIT |
| PubChem periodic table (U.S. National Library of Medicine) | ground truth for element claims | public domain |
| HotpotQA distractor validation set (Yang et al., 2018) | questions, passages, answers | CC BY-SA 4.0 |

The HotpotQA-derived files (`hotpot_*.csv`) are shared under CC BY-SA 4.0, with attribution to
Yang, Qi, Zhang, Bengio, Cohen, Salakhutdinov and Manning (2018), "HotpotQA: A Dataset for
Diverse, Explainable Multi-hop Question Answering", EMNLP.

Everything else in this folder (the generator notebook and the model outputs) follows the
repository's licence: MIT for code, CC BY 4.0 for text (see the License section of the root
README).

## Files

| File | One row per | Key columns |
|---|---|---|
| `mmlu_questions.csv` | question (2,513) | `qid, subject, question, A, B, C, D, answer` |
| `mmlu_llm_outputs.csv` | question | `{base,rerun,shuffled,altprompt}_p{A..D}`, `*_letter_mass`, `shuffled_order`, `verbal_answer`, `verbal_confidence`, `samples_{A..D}`, `samples_invalid` |
| `pubchem_elements.csv` | element (118) | PubChem columns |
| `element_claims.csv` | claim (1,296) | `element, attribute, sentence, claimed_value, true_value, correct, stated_confidence, resample_agreement` |
| `hotpot_questions.csv` | question (799) | `qid, question, answer, type, level` |
| `hotpot_passages.csv` | passage (10 per question) | `qid, passage_idx, title, text, is_supporting, similarity, rank` |
| `hotpot_answers.csv` | question and depth k in {1, 2, 3, 5, 10} | `qid, k, model_answer, exact_match, f1` |

Column meanings:

- `base_p*` are the probabilities of the four answer letters from the first answer token's
  log-probabilities at temperature 0, renormalized over A-D; `rerun_p*` is the identical request
  sent again; `shuffled_p*` used a shuffled option order and is mapped back to the original
  letters; `altprompt_p*` used a different prompt template.
- One caution on `altprompt_p*`: for 185 questions no answer letter appeared among the 20 most
  likely first tokens under the alternative prompt, and the generator stored a uniform 0.25 for
  each option as a placeholder. Identify them as the rows whose four `altprompt_p*` values are
  all 0.25 (notebook 16 separates them). The other three variants have no placeholders.
  `*_letter_mass` is rounded to six decimals, so a few tiny but real masses also read as 0.
- `verbal_confidence` is the model's own stated confidence (0-100) in its answer.
- `samples_*` count 10 answers sampled at temperature 1.0.
- `correct` in `element_claims.csv` is graded against PubChem with stated tolerances (for example
  5% on melting point, 1% on atomic mass); it is empty where PubChem has no value.
- `resample_agreement` counts how many of 5 profiles resampled at temperature 1.0 gave the same
  value (0-5).
- `similarity` is the cosine similarity of the question and passage embeddings; `rank` 1 is the
  most similar passage.

## Regenerating

You need an Azure OpenAI resource with a chat and an embedding deployment. Set
`AZURE_OPENAI_ENDPOINT` (and optionally `AZURE_OPENAI_CHAT_DEPLOYMENT`,
`AZURE_OPENAI_EMBED_DEPLOYMENT`, `AZURE_OPENAI_API_KEY`; without a key it uses `az login`),
install `pyarrow` (only the generator needs it), then execute the notebook. Requests are cached
in `~/.cache/conformal_course_llm`, so an interrupted run resumes where it stopped.
