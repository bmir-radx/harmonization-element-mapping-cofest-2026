# Harmonization Problems

This directory contains the CoFest harmonization problem. It is based on a
real harmonization task from the **RADx** project, where human study data
collected by four different programs had to be harmonized to a single common
data model — the **Global Code Book (GCB)**.

## The setup

There are two kinds of data dictionaries here:

- **Source dictionaries**
  ([`datadictionaries/source-dictionaries/`](datadictionaries/source-dictionaries/))
  — one per RADx program: **RADx-rad**
  ([`rad`](datadictionaries/source-dictionaries/rad/)), **RADx-UP**
  ([`up`](datadictionaries/source-dictionaries/up/)), **RADx Tech**
  ([`tech`](datadictionaries/source-dictionaries/tech/)), and **RADx DHT**
  ([`dht`](datadictionaries/source-dictionaries/dht/)). Each program was run
  by its own Data Coordination Center (DCC) and collected data in its own way,
  which is exactly why the data had to be harmonized.
- **Target dictionary**
  ([`datadictionaries/target-dictionary/`](datadictionaries/target-dictionary/))
  — the data dictionary for the Global Code Book that all programs were
  harmonized to.

Each dictionary is provided in three formats: CSV (`*.dd.csv`), a LinkML
schema (`*.yaml`), and a browsable HTML rendering (`*.html`) — for example,
the target dictionary comes as
[`gcb.dd.csv`](datadictionaries/target-dictionary/gcb.dd.csv),
[`gcb.yaml`](datadictionaries/target-dictionary/gcb.yaml), and
[`gcb.html`](datadictionaries/target-dictionary/gcb.html). The CSV and
LinkML formats carry the same information and are convertible to each other —
both are provided so you can work with whichever you prefer.

## The task

For each data element in the target dictionary, work out which data element(s)
in each source dictionary map to it. Mappings can be one-to-one or
many-to-one — one or more source elements from a program may harmonize to a
single target element.

## The gold standard

During the RADx project this mapping was done by hand, in an Excel
spreadsheet. We have converted that hand-crafted work into machine-readable
harmonization rules — one set per program — in the
[`gold-standard/`](gold-standard/) directory, available as both JSON and YAML:

- RADx-rad: [`radx_rad_rules.json`](gold-standard/radx_rad_rules.json) /
  [`radx_rad_rules.yaml`](gold-standard/radx_rad_rules.yaml)
- RADx-UP: [`radx_up_rules.json`](gold-standard/radx_up_rules.json) /
  [`radx_up_rules.yaml`](gold-standard/radx_up_rules.yaml)
- RADx Tech: [`radx_tech_rules.json`](gold-standard/radx_tech_rules.json) /
  [`radx_tech_rules.yaml`](gold-standard/radx_tech_rules.yaml)
- RADx DHT: [`dht_rules.json`](gold-standard/dht_rules.json) /
  [`dht_rules.yaml`](gold-standard/dht_rules.yaml)

These are the ground truth: use them to evaluate how well your AI-suggested
mappings recover what the human experts actually decided.
