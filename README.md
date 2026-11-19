# AspectPulse — Aspect-Based Customer Review Analytics with Structured Outputs

**Prompt Engineering Mini Project**

**Problem Statement:** Aspect-Based Customer Review Analytics with Structured Outputs

**Deliverables:** D1 — Structured Extraction, D2 — Prompt Comparison, D3 — Product Summarization and Evaluation

---

## 1. Project Overview

AspectPulse is an LLM-powered customer review analytics system that converts unstructured e-commerce product reviews into validated, structured information.

The system identifies:

- Overall customer sentiment (positive, negative, neutral, or mixed)
- Product aspects and their individual sentiment
- Customer complaint categories
- Supporting evidence directly extracted from reviews
- Product-level summaries highlighting customer satisfaction and improvement areas

The project compares three prompt engineering techniques: **Zero-Shot, Few-Shot, and Chain-of-Thought (CoT)**.

It also implements an **Extract → Aggregate → Summarise** pipeline to generate meaningful product insights.

All LLM inference runs locally using **Ollama**, without requiring API keys or external cloud services.

The application provides an interactive six-page Streamlit dashboard for analysis, experiments, visualization, and evaluation.

## 2. Problem Statement

Develop an LLM pipeline that converts unstructured product reviews into validated structured records containing sentiment, product aspects, and complaint categories, and generates per-product summaries.

The pipeline must use public e-commerce review data and compare prompting techniques for sentiment analysis and multi-label classification.

### Project Deliverables

| Deliverable | Description |
|---|---|
| D1 | Structured extraction using Pydantic, JSON/CSV export, validation, and automatic repair |
| D2 | Comparison of Zero-Shot, Few-Shot, and CoT prompting using Macro-F1 |
| D3 | Extract → Aggregate → Summarise pipeline with LLM-as-a-Judge evaluation |

## 3. Project Objectives

1. Convert unstructured customer reviews into structured records.
2. Identify overall sentiment, product aspects, and complaint categories.
3. Validate generated outputs using Pydantic schemas.
4. Automatically repair malformed or invalid LLM outputs.
5. Compare different prompting techniques using classification metrics.
6. Generate product-level summaries from extracted information.
7. Evaluate summary faithfulness using a separate LLM.
8. Provide an interactive dashboard for exploring results.
9. Maintain a fully local and reproducible inference workflow.

## 4. Technology Stack

| Component | Technology |
|---|---|
| Programming Language | Python 3.11 |
| LLM Runtime | Ollama |
| Extraction Model | Qwen2.5:3B |
| Evaluation Model | Llama3.2 |
| Optional Comparison Model | Qwen2.5:7B |
| LLM Framework | LangChain |
| Structured Output Validation | Pydantic v2 |
| Data Processing | Pandas, NumPy |
| Evaluation Metrics | Scikit-learn |
| User Interface | Streamlit |
| Data Visualization | Plotly |
| Configuration | Python-dotenv |
| Testing | Pytest |

## 5. System Architecture

The complete system follows four major stages:

```text
          CUSTOMER REVIEWS
                 |
                 v
       PROMPT ENGINEERING
     Zero-Shot / Few-Shot / CoT
                 |
                 v
         LOCAL LLM (OLLAMA)
                 |
                 v
        STRUCTURED EXTRACTION
                 |
                 v
         PYDANTIC VALIDATION
                 |
           Valid Output?
            /        \
          Yes         No
           |           |
           |      REPAIR LOOP
           |           |
           +-----------+
                 |
                 v
       STRUCTURED REVIEW DATA
                 |
                 v
         PANDAS AGGREGATION
                 |
                 v
        PRODUCT SUMMARIZATION
                 |
                 v
          LLM-AS-A-JUDGE
                 |
                 v
        STREAMLIT DASHBOARD
```

### Architecture Explanation

**Stage 1 — Extraction**

The system reads customer reviews and uses an LLM to identify sentiment, aspects, and complaints.

**Stage 2 — Validation**

Pydantic checks whether the extracted information follows the required schema. Invalid outputs are repaired or recorded as controlled failures.

**Stage 3 — Aggregation and Summarization**

Pandas calculates product statistics. The LLM then converts these statistics and supporting review evidence into readable summaries.

**Stage 4 — Evaluation**

Classification metrics measure extraction performance, while a separate LLM evaluates summary faithfulness and correctness.

## 6. Dataset

### Dataset Source

**Amazon Reviews 2023 — Appliances Category**

Dataset website: https://amazon-reviews-2023.github.io/

The dataset is published by the McAuley Lab and contains real-world e-commerce product reviews.

### Dataset Preparation

The original Appliances category contains approximately **2.12 million reviews**.

The preprocessing pipeline performs the following operations:

1. Cleans and normalizes review text.
2. Filters reviews containing 5–400 words.
3. Selects products with sufficient valid reviews.
4. Removes duplicate reviews.
5. Selects 15 products using a fixed random seed.
6. Samples 100 reviews from each product.

### Final Dataset

| Property | Value |
|---|---|
| Original Reviews | 2,128,605 |
| Selected Products | 15 |
| Reviews per Product | 100 |
| Total Processed Reviews | 1,500 |
| Evaluation Sample | 300 |
| Evaluable Reference Reviews | 299 |
| Random Seed | 42 |

Star ratings are used for sampling and display purposes, but are not provided to the LLM during sentiment extraction.

## 7. Annotation Methodology

The evaluation sample contains 300 reviews selected using rating-stratified sampling.

Each review is labelled with:

- Overall sentiment
- Aspect categories
- Aspect-level sentiment
- Complaint categories

### Reference Annotation Details

| Annotation Type | Count |
|---|---:|
| Human-labelled reference records | 1 |
| AI-assisted reference records | 299 |
| Excluded unreadable records | 1 |
| Evaluable reference records | 299 |

The AI-assisted reference labels were generated using Claude from review text, with mentor approval.

An additional independent blind human annotation set containing 300 completed records is maintained separately.

**Important:** The reported D2 results use the original 299 evaluable reference labels. Since most are AI-assisted, these metrics measure agreement with the reference labels rather than independently verified human accuracy.

## 8. Structured Extraction Pipeline — D1

Deliverable 1 focuses on converting raw review text into reliable structured data.

### Extraction Process

```text
Raw Review
    |
    v
Prompt Construction
    |
    v
LLM Response
    |
    v
JSON Parsing
    |
    v
Pydantic Validation
    |
    v
Evidence Grounding
    |
    v
Automatic Repair (if needed)
    |
    v
Validated ReviewRecord
    |
    v
JSON / CSV Export
```

### Pydantic Schema

The structured output includes:

| Field | Description |
|---|---|
| review_id | Unique review identifier |
| product_id | Product identifier |
| review_text | Original review |
| overall_sentiment | Positive, negative, neutral, or mixed |
| aspects | Extracted product aspects |
| complaint_categories | Identified customer complaints |
| confidence | Model confidence between 0 and 1 |
| rationale | Optional short explanation |
| extraction_metadata | Model, strategy, validation, repair, and runtime details |

### Example Structured Output

Input review:

> "The washing machine works well and is easy to use, but it makes too much noise."

Example output (simplified):

```json
{
  "review_id": "R001",
  "product_id": "P001",
  "overall_sentiment": "mixed",
  "aspects": [
    {
      "aspect_name": "ease of use",
      "aspect_category": "ease_of_use",
      "sentiment": "positive",
      "evidence": "easy to use",
      "confidence": 0.95
    },
    {
      "aspect_name": "noise",
      "aspect_category": "noise",
      "sentiment": "negative",
      "evidence": "makes too much noise",
      "confidence": 0.96
    }
  ],
  "complaint_categories": [
    "noise_issue"
  ],
  "confidence": 0.94
}
```

The actual exported records also include the configured extraction metadata and other schema fields.

### Automatic Output Repair

LLMs sometimes produce malformed JSON or incorrect field values.

The repair pipeline:

1. Captures the raw LLM output.
2. Extracts JSON from surrounding text or code fences.
3. Applies safe formatting corrections.
4. Normalizes permitted formatting differences.
5. Validates the result with Pydantic.
6. Sends validation errors back to the LLM when necessary.
7. Retries up to two times.
8. Records a controlled failure if validation still fails.

The system also checks that extracted evidence matches the original review.

### Export Formats

- Structured JSON records
- Review-level CSV records
- Aspect-level CSV records
- Failed extraction records
- Validation and repair reports

### Main D1 Files

```text
schemas/review_schema.py
schemas/taxonomy.py
pipeline/extraction.py
pipeline/validation.py
pipeline/repair.py
utils/export.py
scripts/export_d1.py
```

## 9. Prompt Engineering Techniques — D2

Three prompting techniques are compared using the same dataset, model, output schema, and evaluation settings.

### Zero-Shot Prompting

The LLM receives instructions and the customer review without examples.

The model must identify sentiment, aspects, and complaints using the supplied definitions.

### Few-Shot Prompting

The LLM receives five fixed synthetic examples demonstrating the required output.

These examples include positive, negative, mixed, neutral, and compatibility-related reviews.

The examples are not taken from the evaluation dataset.

### Chain-of-Thought Prompting

The model is guided through a structured seven-step analysis procedure:

1. Identify important review statements.
2. Detect product aspects.
3. Assign aspect categories.
4. Determine sentiment for each aspect.
5. Identify complaint categories.
6. Determine overall sentiment.
7. Verify evidence against the review.

The model performs this analysis internally and returns only the structured JSON and a concise rationale, not a detailed reasoning trace.

### Prompt Comparison

| Technique | Main Approach |
|---|---|
| Zero-Shot | Instructions without examples |
| Few-Shot | Instructions with five demonstrations |
| Chain-of-Thought | Instructions with a structured analysis procedure |

## 10. Evaluation Methodology

The three prompting strategies are evaluated against the same reference labels.

The primary evaluation metric is **Macro-F1 Score**.

### Why Macro-F1?

Accuracy measures the proportion of correct predictions, but it may hide poor performance on less frequent sentiment classes.

Macro-F1 calculates F1 separately for every class and gives each class equal importance.

**Precision**

```text
Precision = TP / (TP + FP)
```

**Recall**

```text
Recall = TP / (TP + FN)
```

**F1 Score**

```text
F1 = 2 × Precision × Recall
     ----------------------
      Precision + Recall
```

**Macro-F1**

```text
Macro-F1 = Average F1 across all classes
```

### Additional Evaluation Metrics

- Accuracy
- Macro Precision and Recall
- Weighted F1
- Per-class Precision, Recall, and F1
- Confusion Matrix
- Multi-label Complaint F1
- Aspect Detection F1
- Aspect Sentiment F1
- Validity Rate
- Repair Rate
- Extraction Coverage
- Processing Latency
- Token Usage

Failed extractions remain part of the evaluation denominator and are not silently excluded.

### Experimental Results

The final D2 scores are generated from completed local experiment runs and stored in the `reports/` directory.

Results must be taken from verified experiment artifacts rather than mock runs or incomplete executions.

## 11. Product Summarization Pipeline — D3

Deliverable 3 implements an **Extract → Aggregate → Summarise** prompt chain.

### Step 1 — Extract

Customer reviews are converted into structured records containing sentiment, aspects, and complaints.

### Step 2 — Aggregate

Pandas combines the extracted information for each product.

It calculates:

- Total analyzed reviews
- Positive and negative sentiment percentages
- Frequently mentioned aspects
- Aspect-level sentiment distributions
- Common customer complaints
- Complaint percentages
- Product sentiment trends
- Supporting review evidence

All numerical statistics are computed programmatically rather than generated by the LLM.

### Step 3 — Summarise

The LLM receives aggregated statistics and supporting review evidence.

It generates a product summary containing:

- Overall customer opinion
- Most appreciated features
- Frequently reported complaints
- Commonly discussed aspects
- Suggested improvement areas
- Product recommendations

Summary claims are linked to review identifiers for traceability.

## 12. LLM-as-a-Judge

The generated product summaries are evaluated using a separate local model.

| Purpose | Model |
|---|---|
| Review Extraction | Qwen2.5:3B |
| Product Summarization | Qwen2.5:3B |
| Summary Evaluation | Llama3.2 |

Using a different model family for evaluation helps reduce direct self-preference.

### Evaluation Criteria

The original judge evaluates:

1. **Faithfulness:** Whether claims are supported by reviews.
2. **Coverage:** Whether important product information is included.
3. **Consistency:** Whether statements agree with the extracted statistics.
4. **Hallucination:** Whether the summary introduces unsupported information.

The original rubric uses a 1–5 scale.

### Final Claim-Level Judge — Version 4

The final evaluation framework uses a more detailed claim-level approach.

Each generated summary is divided into individual claims.

Claims are classified as:

- SUPPORTED
- PARTIALLY_SUPPORTED
- UNSUPPORTED
- CONTRADICTED
- UNVERIFIABLE

The evaluation system combines deterministic checks with LLM verification.

It checks numerical claims, review citations, sentiment consistency, and supporting evidence.

Final scores are calculated from claim statuses using fixed rules.

The judge is also tested against a constructed negative-control dataset containing deliberately incorrect claims.

**Note:** LLM-as-a-Judge results are automated assessments and should not be treated as independently verified human judgments.

## 13. Streamlit Dashboard

AspectPulse includes six interactive dashboard pages.

| Page | Functionality |
|---|---|
| Overview | Dataset statistics, project status, and visualizations |
| Review Analysis | Review extraction, structured output, and repair diagnostics |
| Prompt Experiments | Comparison of prompting strategies and evaluation metrics |
| Product Intelligence | Product sentiment, aspects, complaints, and summaries |
| Summary Evaluation | Judge results, claim verification, and faithfulness checks |
| Human Annotation | Annotation review, validation, and independent labelling |

The dashboard uses Streamlit and Plotly to provide interactive visualizations.

Live model outputs and mock demonstration outputs are stored separately to prevent confusion.

## 14. Project Structure

```text
AspectPulse/
│
├── app.py
├── config.py
├── requirements.txt
├── .env.example
│
├── schemas/
│   ├── review_schema.py
│   ├── taxonomy.py
│   └── evaluation_schema.py
│
├── llm/
│   ├── base.py
│   ├── factory.py
│   ├── langchain_client.py
│   └── mock.py
│
├── prompts/
│   ├── common.py
│   ├── zero_shot.py
│   ├── few_shot.py
│   ├── cot.py
│   ├── strategies.py
│   ├── repair.py
│   ├── summarization.py
│   └── judge.py
│
├── pipeline/
│   ├── extraction.py
│   ├── validation.py
│   ├── repair.py
│   ├── aggregation.py
│   └── summarization.py
│
├── evaluation/
│   ├── classification_metrics.py
│   ├── prompt_comparison.py
│   ├── summary_evaluation.py
│   ├── claims.py
│   ├── claim_checks.py
│   └── judge_v4.py
│
├── annotation/
│   ├── annotation_manager.py
│   └── annotation_guide.md
│
├── ui/
│   └── Dashboard Pages
│
├── scripts/
│   ├── prepare_dataset.py
│   ├── build_annotation_sample.py
│   ├── run_experiment.py
│   ├── run_chain.py
│   ├── export_d1.py
│   ├── export_d3.py
│   └── build_reports.py
│
├── data/
│   ├── raw/
│   ├── processed/
│   └── annotations/
│
├── outputs/
│   ├── live/
│   └── mock/
│
├── reports/
├── docs/
├── assets/
└── tests/
```

## 15. Installation and Setup

### Prerequisites

Ensure the following are installed:

- Python 3.11
- Pip
- Ollama
- Git (optional for development)

### Step 1 — Open the Project Directory

Open a terminal in the existing project folder.

### Step 2 — Create a Virtual Environment

```bash
python -m venv .venv
```

Activate the virtual environment.

**Windows**

```bash
.venv\Scripts\activate
```

**macOS / Linux**

```bash
source .venv/bin/activate
```

### Step 3 — Install Dependencies

```bash
pip install -r requirements.txt
```

### Step 4 — Prepare the Dataset

```bash
python scripts/prepare_dataset.py
```

This downloads and processes the required Amazon Reviews dataset.

### Step 5 — Configure Environment Variables

**Windows**

```bash
copy .env.example .env
```

**macOS / Linux**

```bash
cp .env.example .env
```

The default configuration uses local Ollama inference and does not require an API key.

## 16. Ollama Setup

Install Ollama from:

https://ollama.com/

Download the required models:

```bash
ollama pull qwen2.5:3b
ollama pull llama3.2
```

Optional model for comparison:

```bash
ollama pull qwen2.5:7b
```

Verify the installation:

```bash
ollama list
```

Check the local server:

```bash
curl http://127.0.0.1:11434/api/version
```

### Environment Configuration

```env
LLM_PROVIDER=ollama
LLM_MODEL=qwen2.5:3b

JUDGE_PROVIDER=ollama
JUDGE_MODEL=llama3.2

OLLAMA_BASE_URL=http://127.0.0.1:11434

OLLAMA_NUM_CTX=8192
LLM_TIMEOUT_SECONDS=300
LLM_MAX_OUTPUT_TOKENS=1024
LLM_TEMPERATURE=0

MAX_REPAIR_ATTEMPTS=2
STRICT_EVIDENCE_GROUNDING=true
```

The application rejects non-local Ollama endpoints by default.

## 17. Running the Application

Start the Streamlit dashboard:

```bash
streamlit run app.py
```

Open the application at:

http://localhost:8501

The dashboard provides access to review analysis, prompt experiments, product insights, summary evaluation, and annotation tools.

## 18. Running the Experiments

### Run D2 Prompt Comparison

```bash
python scripts/run_experiment.py --only-annotated
```

### Resume an Interrupted Experiment

```bash
python scripts/run_experiment.py --resume <run_id> --only-annotated
```

### Recalculate Evaluation Metrics

```bash
python scripts/run_experiment.py --reevaluate <run_id>
```

### Export D1 Structured Records

```bash
python scripts/export_d1.py --run <run_id>
```

### Run D3 Extraction

```bash
python scripts/run_chain.py --per-product 20 --extract-only
```

### Generate Product Summaries and Judge Results

```bash
python scripts/run_chain.py --per-product 20 --judge
```

### Run Final Claim-Level Judge

```bash
python scripts/judge_v4.py run --source dataset__few_shot__qwen2.5-3b__v1 --objects original nc corrected
```

### Generate Judge Report

```bash
python scripts/judge_v4.py report --source dataset__few_shot__qwen2.5-3b__v1
```

### Run Automated Tests

```bash
python -m pytest -q
```

## 19. Reliability and Reproducibility

AspectPulse includes several mechanisms to improve reliability.

**Structured Validation**

All LLM outputs are checked against predefined Pydantic schemas.

**Evidence Grounding**

Aspect evidence must match the original review text.

**Automatic Repair**

Malformed outputs are repaired through deterministic corrections and limited LLM retries.

**Resumable Execution**

Completed extraction results are cached so interrupted experiments can continue without restarting.

**Controlled Failures**

Invalid responses and provider outages are recorded separately.

**Reproducible Experiments**

Experiment settings, prompt versions, model configurations, predictions, and validation logs are stored for later verification.

**Local Inference**

Default inference runs on the local machine using Ollama.

## 20. Project Limitations

1. Most D2 reference labels are AI-assisted, so evaluation scores are not equivalent to independently verified human accuracy.
2. Smaller local LLMs may miss important product aspects or complaints.
3. Larger models require more computational resources and processing time.
4. Some generated responses may exceed output limits and require repair.
5. Aspect matching primarily uses predefined categories.
6. The evaluation sample does not represent the original dataset's natural rating distribution.
7. LLM-as-a-Judge evaluations may contain errors and are not a substitute for human assessment.
8. Product summaries are based on the available evaluation sample rather than every review in the original dataset.

## 21. Future Improvements

Potential improvements include:

- Expanding independent human-labelled evaluation data
- Measuring inter-annotator agreement
- Using larger local language models
- Implementing constrained JSON generation
- Improving aspect detection using embeddings
- Introducing dynamic few-shot example selection
- Improving confidence calibration
- Comparing LLM judge results with human assessments
- Optimizing inference speed and memory usage

## 22. Team Contributions

### Piyush Nadgir

**Lead Architecture, Structured Extraction Pipeline, and User Interface**

**Primary Responsibility: Deliverable 1 and Application Platform**

Contributions:

- Designed the overall project architecture and structured extraction workflow.
- Developed Pydantic schemas for review records, product aspects, and complaint categories.
- Implemented the review extraction and validation pipeline.
- Developed the automatic JSON repair mechanism for malformed LLM responses.
- Implemented evidence-grounding checks and controlled failure handling.
- Developed JSON and CSV export functionality.
- Built the six-page Streamlit analytics dashboard.
- Implemented dashboard navigation, visualization, and interface styling.
- Integrated the structured extraction pipeline with the application.

**Key Modules**

```text
schemas/review_schema.py
schemas/taxonomy.py

pipeline/extraction.py
pipeline/validation.py
pipeline/repair.py

utils/export.py

app.py
ui/
assets/
.streamlit/
```

### Ramneek Gautham

**Prompt Engineering, Comparative Evaluation, and Product Intelligence**

**Primary Responsibility: Deliverables 2 and 3**

Contributions:

- Designed Zero-Shot, Few-Shot, and Chain-of-Thought prompting strategies.
- Developed fixed synthetic few-shot examples.
- Implemented the comparative evaluation workflow for the 300-review sample.
- Evaluated sentiment classification using Macro-F1 and additional metrics.
- Developed the Extract → Aggregate → Summarise prompt chain.
- Implemented product-level aggregation and summarization.
- Developed the LLM-as-a-Judge evaluation framework.
- Implemented summary faithfulness and grounding checks.
- Contributed to claim-level evaluation and product summary validation.

**Key Modules**

```text
prompts/zero_shot.py
prompts/few_shot.py
prompts/cot.py
prompts/summarization.py
prompts/judge.py

pipeline/aggregation.py
pipeline/summarization.py

evaluation/
```

## 23. References

**Dataset**

Hou et al. (2024), *Bridging Language and Items for Retrieval and Recommendation*, Amazon Reviews 2023.

https://arxiv.org/abs/2403.03952

**Technologies and Documentation**

- Ollama: https://ollama.com/
- LangChain: https://python.langchain.com/
- Pydantic: https://docs.pydantic.dev/
- Streamlit: https://streamlit.io/
- Scikit-learn: https://scikit-learn.org/
- Pandas: https://pandas.pydata.org/

---

**AspectPulse — Aspect-Based Customer Review Analytics with Structured Outputs**

*Prompt Engineering Mini Project | Deliverables D1, D2, and D3*
