# Mapping Data Elements with AI

**BOSC 2026 Collaboration Fest (CoFest) project**

 AI-assisted suggestion of mappings between **source** and **target** data dictionaries, to make biomedical data harmonization faster with humans in the loop.

This project runs **alongside our CoFest poster** on the
[Harmonizer](#the-tool-were-building-around) tool being developed at Stanford Computational Medicine. Come to the poster, then come build with us.


---

## Summary

Harmonizing biomedical data means manually deciding which variable in a *source*
dataset corresponds to which variable in a *target* data model, and how to
transform the values. It's slow, hard to scale, and needs domain experts.

In this CoFest we'll prototype **AI that suggests candidate mappings** between
source and target data elements, using the metadata that's already in the data
dictionaries — variable names, descriptions, permissible values, data types,
units, and context. We are **not** trying to fully automate harmonization. The
goal is to generate plausible suggestions that a human can review, refine, and
accept.

---

## The problem

Given two data dictionaries — one describing the *source* data, one describing
the *target* model — a curator today reads each variable and manually matches
them. For example, deciding that the source variable `current_employment_status`
should map to the target variable `nih_employment`, and that the answer codes
need to be remapped (`enum_to_enum`), is something a person currently does by
hand, one variable at a time.

That doesn't scale to hundreds of variables, and evolving target models. **The metadata to make good guesses is already sitting in the
dictionaries, we just aren't using it yet.**

---

## What we'll build at CoFest

A pipeline (or several competing prototypes) that:

1. Reads a **source dictionary** and a **target dictionary**.
2. Suggests, for each source element, the most likely **target element(s)** it
   maps to — with a confidence score and a short rationale.
3. Optionally suggests the **transformation** needed (e.g. unit conversion,
   value-code remapping), expressed as the framework's rule format.
4. Presents suggestions for **human review** rather than applying them blindly.

The mapping is **not blind**: every suggestion is grounded in the rich metadata
the dictionaries provide.

---

## The data: what the AI gets to work with

Data dictionaries are CSVs. Each row is one data element. The columns are the
signal your model should use:

| Column | What it gives the model |
| --- | --- |
| `Id` | The variable name (e.g. `commute_distance_miles`) |
| `Label` | A human description / question text |
| `Terms` | Ontology / vocabulary terms, where available |
| `Datatype` | `int`, `float`, `string`, … |
| `Pattern` | Expected format / regex |
| `Unit` | e.g. `miles`, `fahrenheit`, `kelvin` |
| `Enumeration` | Permissible values + their labels (e.g. `"1"=[Working now] | "4"=[Retired] …`) |
| `Additional Missing Value Codes` | e.g. `"-9999"=[Reason Unknown]` |
| `Notes` | Free-text context |

### A concrete example

A **source** dictionary and a **target** dictionary, side by side:

```
SOURCE (demo_dictionary1.csv)              TARGET (demo_dictionary_target.csv)
─────────────────────────────             ──────────────────────────────────
current_employment_status (int, enum)  →  nih_employment (int, enum)
commute_distance_miles    (float, mi)  →  commute_distance_miles (float, mi)
temperature_f             (float, °F)  →  temperature_k (float, K)
```

The mappings a human would confirm here:

- `current_employment_status → nih_employment` — same concept, **different value
  codes** → needs an `enum_to_enum` remap.
- `commute_distance_miles → commute_distance_miles` — identical → no-op.
- `temperature_f → temperature_k` — same concept, **different unit** → needs a
  `convert_units` transform.

These are exactly the calls we want AI to *propose*, so a curator only has to
*verify*.

### What a confirmed mapping looks like (the output format)

The harmonization framework consumes mappings as **rules**. Each rule names the
source(s), the target, and an ordered list of operations:

```json
[
  {
    "sources": ["temperature_f"],
    "target": "temperature_k",
    "operations": [
      { "operation": "convert_units", "source_unit": "fahrenheit", "target_unit": "kelvin" }
    ]
  },
  {
    "sources": ["current_employment_status"],
    "target": "nih_employment",
    "operations": [
      { "operation": "enum_to_enum",
        "mapping": { "1": "0", "3": "1", "4": "4", "6": "3", "7": "2" },
        "default": "97", "strict": false }
    ]
  }
]
```

So a "mapping suggestion" can range from just the `sources → target` pairing
(the core task) all the way to a fully-formed rule with operations (a stretch
goal).

---

## Project ideas / tracks

Pick what matches your interests — these can run in parallel:

- **LLM-based mapping suggestions.** Feed source + target element metadata to an
  LLM and get back ranked candidate targets + a rationale, automatically — no
  manual prompting in the loop. Explore embeddings vs. generative matching and
  structured output.
- **Hybrid AI + rule-based.** Combine cheap rules (datatype/unit/name overlap)
  with LLM/embedding scoring; rules prune, AI ranks.
- **Benchmark datasets & evaluation.** Build labelled source→target pairs and a
  metric (precision@k, top-1 accuracy, time saved) so we can compare approaches
  objectively.
- **Human-in-the-loop interface.** A lightweight UI to review, accept, reject,
  and edit suggestions — feeding into (or extending) the Harmonizer app.

---

## The tool we're building around

**Harmonizer** is an open-source desktop app for transparent, reproducible,
fully-offline data harmonization. Researchers load a source dataset plus source
and target data dictionaries, map the elements, and run transformations locally.

- **Frontend (Angular + Electron):**
  https://github.com/bmir-radx/harmonization-ui-angular
- **Backend / framework (Python + FastAPI "sidecar"):**
  https://github.com/bmir-radx/harmonization-framework
- **Worked examples & tutorials:**
  https://bmir-radx.github.io/harmonization-examples/

At runtime the Electron frontend spawns the compiled Python sidecar and proxies
UI interactions to it. Our CoFest work plugs into the **mapping** step: today a
human draws those source→target connections; we want AI to propose them first.

---

## Prerequisites (read / skim before the CoFest)

You'll get up to speed fastest in this order:

1. **Skim the examples** — the fastest way to understand mappings, rules, and
   operations: https://bmir-radx.github.io/harmonization-examples/
   Start with `01-hello-harmonization`, then `02-units-and-numbers`.
2. **Look at the rule model** — `sources / target / operations` and the
   primitives table (`enum_to_enum`, `convert_units`, `scale`, `bin`, …) in the
   framework README:
   https://github.com/bmir-radx/harmonization-framework#primitives-reference
3. **Open a pair of dictionaries** — the demo files
   `demo/demo_dictionary1.csv` (source) and `demo/demo_dictionary_target.csv`
   (target) in the framework repo are our starting test case.

Optional: run the Harmonizer app from the
[Releases page](https://github.com/bmir-radx/harmonization-ui-angular/releases)
to see the human workflow we're augmenting.

---

## Getting started (run the framework locally)

```bash
# Backend / framework
git clone https://github.com/bmir-radx/harmonization-framework.git
cd harmonization-framework
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
pip install .

# Run the demo harmonization end-to-end
cd demo/harmonize_example
python run_example.py        # applies rules.json to input.csv -> output.csv
```

Then try the CLI on the demo data:

```bash
harmonize \
  --rules demo/harmonize_example/rules.json \
  --input demo/harmonize_example/input.csv \
  --output harmonized.csv \
  --on-missing warn
```

Once you can run a *known* mapping, you're ready to start *suggesting* mappings.

---

## Resources

- Frontend: https://github.com/bmir-radx/harmonization-ui-angular
- Backend / framework: https://github.com/bmir-radx/harmonization-framework
- Examples & tutorials: https://bmir-radx.github.io/harmonization-examples/

## License

[GPL-3.0](https://github.com/bmir-radx/harmonization-element-mapping-cofest-2026/blob/main/LICENSE)

## Acknowledgements

Developed in the context of a data harmonization framework and tool at Stanford
University (Computational Medicine / RADx).
