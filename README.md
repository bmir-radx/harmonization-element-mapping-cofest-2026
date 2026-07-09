# Mapping Data Elements with AI

**BOSC 2026 Collaboration Fest (CoFest) project**

This repository contains the problem set and documentation for our CoFest project. The project focuses on using AI/LLM methods to suggest mapping candidates between source data elements and target data elements.

---

## Project Description

The goal of this project is to develop methods that **suggest candidate mappings** from source data elements to target data elements.

Given a **source** data dictionary and a **target** data dictionary (the Global Code Book), your solution should identify which source data element(s) map to each target data element.

### What is Data Harmonization?

In biomedical research, different projects collect data using custom schemas, data elements, and formats. To analyze data together, curators must manually map data elements representing the same clinical concept to a unified schema.

For example, a study might record education levels as `edu_years_of_school` (with numeric year categories), whereas the target model requires `nih_education`. A human curator uses the [Harmonizer](https://github.com/bmir-radx/harmonization-ui-angular) desktop app to build and run mapping rules.

Here is a visual screenshot of how the Harmonizer interface displays these mappings and suggests connections:

![Harmonizer UI Desktop Screenshot](assets/harmonizer_ui.png)

#### 📋 Component Definitions & Examples

1. **Source Data Element**
   - **Definition**: A data element and its associated metadata collected by an individual research program.
   - **Example**: `edu_years_of_school` from the RADx-UP program.
     - *Metadata*:
       - **Label**: "What is the highest level of education you have achieved outside or in the United States? Grades roughly equivalent to years of school."
       - **Enumerations**: `"0"=[No formal schooling] | "4"=[High school graduate or GED completed] | "5"=[Some college level...]`
       - **Datatype**: `integer`

2. **Target Data Element**
   - **Definition**: A standardized data element in the common destination schema (the Global Code Book).
   - **Example**: `nih_education` in the Global Code Book (GCB).
     - *Metadata*:
       - **Label**: "Education Level"
       - **Datatype**: `integer`
       - **Description**: Standardized codebook categorization for education.

3. **Harmonization Mapping Rule**
   - **Definition**: The configuration mapping the source data element(s) to the target data element.
   - **Example mapping rule** (simplified to focus on the mapping only, ignoring transformations):

     ```json
     {
       "sources": ["edu_years_of_school"],
       "target": "nih_education",
       "metadata": {
         "program": "RADx-UP",
         "concept": "Education Level"
       }
     }
     ```

---

## The CoFest Project: RADx Problem Set

The problem set is based on a real harmonization task from the NIH **RADx** (Rapid Acceleration of Diagnostics) project, where human study data collected by four different programs had to be harmonized to a single common data model — the **Global Code Book (GCB)**.

The [problems/](problems/) directory contains the real-world problem set:

- 📂 **[Source Data Dictionaries](problems/datadictionaries/source-dictionaries/)**: Data Elements describing data collected by four NIH programs:
  - [RADx-rad](problems/datadictionaries/source-dictionaries/rad/)
  - [RADx-UP](problems/datadictionaries/source-dictionaries/up/)
  - [RADx Tech](problems/datadictionaries/source-dictionaries/tech/)
  - [RADx DHT](problems/datadictionaries/source-dictionaries/dht/)
- 📂 **[Target Data Dictionary](problems/datadictionaries/target-dictionary/)**: The target data dictionary for the [Global Code Book (GCB)](problems/datadictionaries/target-dictionary/gcb.dd.csv).
- 📂 **[Gold Standard Mappings](problems/gold-standard/)**: Human-curated mapping rules (ground truth) in JSON/YAML. Each rule contains a `sources` list and a `target` name. *(Note: The gold standard files also contain `operations` for data transformation, but you can ignore those and focus purely on the `sources -> target` mappings).*

Refer to the [Problems README](problems/Problems_README.md) for full details on the project setup and files.

### Dictionary Formats Available

Each dictionary is provided in three formats for flexibility:

- **CSV (`*.dd.csv`)**: Follows the [RADx Data Dictionary Specification](https://github.com/bmir-radx/radx-data-dictionary-specification/blob/main/radx-data-dictionary-specification.md) (defining columns and metadata layout).
- **LinkML Schema (`*.yaml`)**: Machine-readable LinkML model schema.
- **Browsable HTML (`*.html`)**: Interactive page showing metadata annotations and definitions.

### Metadata Signals in CSV dictionaries

The CSV dictionaries include key columns to utilize for your models:

- **`Id`**: Data element name (e.g., `commute_distance_miles`).
- **`Label`**: Human-readable question text / description.
- **`Enumeration`**: Allowed values & labels (e.g., `"1"=[Working now] | "4"=[Retired]`).
- **`Datatype` & `Unit`**: Data type (integer, float, string) and units (e.g., `miles`, `fahrenheit`).
- **`Terms`**: Ontology mapping terms, if available.
- **`Notes`**: Free-text context.

---

## Expectations & Output Format

### What you need to produce

Your mapping tool or pipeline should read the source and target data dictionaries and output a mapping rules file for each source program (e.g., `radx_up_mappings.json`).

### Expected Output Schema

The mapping rules file should be a flat JSON or YAML array of rule entries. Each entry must define the `sources` array and the `target` name.

```json
[
  {
    "sources": ["edu_years_of_school"],
    "target": "nih_education"
  },
  {
    "sources": ["covid_fatique"],
    "target": "nih_fatigue"
  }
]
```

To validate your generated output rules for syntax, schema compliance, and matching targets, you can use the `harmonize --validate` command from the [harmonization-framework](https://github.com/bmir-radx/harmonization-framework):

```bash
harmonize --validate --rules path/to/your_generated_rules.json
```

---

## Project Ideas

Feel free to choose one or more of the following ideas or propose your own:

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
