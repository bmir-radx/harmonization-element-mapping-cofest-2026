# Mapping Data Elements with AI

**BOSC 2026 Collaboration Fest (CoFest) project**

AI-assisted suggestion of mappings between **source** and **target** data dictionaries to make biomedical data harmonization faster with humans in the loop.

This project runs **alongside our CoFest poster** on the [Harmonizer](https://github.com/bmir-radx/harmonization-ui-angular) tool being developed at the Stanford Division of Computational Medicine.

---

## Project Description

The goal of this project is to develop methods that **suggest candidate mappings** from source variables to target variables.

Given a **source** data dictionary and a **target** model dictionary (the Global Code Book), your tool should identify which source data element(s) map to each target data element.

> [!NOTE]
> In this CoFest challenge, we only care about mapping variables together. The underlying data transformation operations (like unit conversion or enum-to-enum remapping) are not the concern of this track.

### Dataset Structure

The [problems/](problems/) directory contains the real-world dataset:

- 📂 **[Source Dictionaries](problems/datadictionaries/source-dictionaries/)**: Metadata for elements collected by four programs:
  - [RADx-rad](problems/datadictionaries/source-dictionaries/rad/)
  - [RADx-UP](problems/datadictionaries/source-dictionaries/up/)
  - [RADx Tech](problems/datadictionaries/source-dictionaries/tech/)
  - [RADx DHT](problems/datadictionaries/source-dictionaries/dht/)
- 📂 **[Target Dictionary](problems/datadictionaries/target-dictionary/)**: The target data dictionary for the [Global Code Book (GCB)](problems/datadictionaries/target-dictionary/gcb.dd.csv).
- 📂 **[Gold Standard Mappings](problems/gold-standard/)**: Human-curated mapping rules (ground truth) in JSON/YAML. Each rule contains a `sources` list and a `target` name. *(Note: The gold standard files also contain `operations` for data transformation, but you can ignore those and focus purely on the `sources -> target` mappings).*

Refer to the [Problems README](problems/Problems_README.md) for full details on the challenge setup and files.

### Metadata Signals Available

The CSV dictionaries follow the [RADx Data Dictionary Specification](https://github.com/bmir-radx/radx-data-dictionary-specification/blob/main/radx-data-dictionary-specification.md) and include key signals for your models:

- **`Id`**: Variable name (e.g., `commute_distance_miles`).
- **`Label`**: Human-readable question text / description.
- **`Enumeration`**: Allowed values & labels (e.g., `"1"=[Working now] | "4"=[Retired]`).
- **`Datatype` & `Unit`**: Data type (integer, float, string) and units (e.g., `miles`, `fahrenheit`).
- **`Terms`**: Ontology mapping terms, if available.
- **`Notes`**: Free-text context.

---

## Project Tracks / Ideas

Feel free to choose a track or propose your own:

- **LLM / Embedding pipeline**: Build an automated pipeline that reads source + target dictionary metadata, queries an LLM/embedding model, and returns ranked candidate mappings with a short rationale.
- **Hybrid Rule + AI matching**: Combine simple heuristic rules (like keyword matching or name overlaps) to prune the candidate space, then use LLMs/embeddings to rank the final suggestions.
- **Evaluation & Benchmarking**: Write evaluation scripts to test your pipeline's predictions against the provided [Gold Standard Mappings](problems/gold-standard/) to compute performance metrics (e.g., accuracy, Precision@K).
- **Human-in-the-loop UI**: Design a lightweight interface to review, accept/reject, or edit mapping suggestions.

---

## Setup & Getting Started

### 1. Prerequisites (Read/Skim)

- **Skim the tutorials**: Read [01-hello-harmonization](https://bmir-radx.github.io/harmonization-examples/) to understand the core rule model.
- **Browse the Dictionaries**: Open the target dictionary [gcb.dd.csv](problems/datadictionaries/target-dictionary/gcb.dd.csv) and one of the sources (e.g., [up.dd.csv](problems/datadictionaries/source-dictionaries/up/up.dd.csv)) to see what columns are available.

### 2. Run the Harmonization Framework Locally

To parse, validate, and execute mappings, you can use the backend harmonization-framework sidecar.

> [!IMPORTANT]
> Requires Python 3.9 to 3.13. Python 3.14+ is not currently supported due to dependency requirements.

```bash
# Clone and install the framework sidecar
git clone https://github.com/bmir-radx/harmonization-framework.git
cd harmonization-framework
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
pip install .

# Test the framework using the CLI on the demo files in the framework repo
harmonize \
  --rules demo/harmonize_example/rules.json \
  --input demo/harmonize_example/input.csv \
  --output harmonized.csv \
  --on-missing warn
```

---

## Resources

- **Angular + Electron UI**: [harmonization-ui-angular](https://github.com/bmir-radx/harmonization-ui-angular)
- **Python Backend Framework**: [harmonization-framework](https://github.com/bmir-radx/harmonization-framework)
- **Interactive Tutorials**: [harmonization-examples](https://bmir-radx.github.io/harmonization-examples/)

---

## License & Acknowledgements

This project is licensed under [GPL-3.0](LICENSE).

Developed in the context of the data harmonization framework and tool at the Stanford University Division of Computational Medicine.
