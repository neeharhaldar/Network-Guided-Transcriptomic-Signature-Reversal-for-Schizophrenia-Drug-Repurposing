# Network-Guided Transcriptomic Signature Reversal for Schizophrenia Drug Repurposing

A systems biology pipeline integrating differential expression analysis, protein–protein interaction (PPI) networks, and transcriptomic signature reversal to identify candidate repurposed therapeutics for schizophrenia.

---

# Overview

Schizophrenia is a complex neuropsychiatric disorder associated with widespread transcriptomic, synaptic, metabolic, and neuroimmune dysregulation. Traditional drug discovery approaches remain costly and time-intensive, motivating the development of computational drug repurposing frameworks.

This project implements a network-guided transcriptomic reversal workflow using publicly available postmortem brain transcriptomic data to identify compounds capable of reversing schizophrenia-associated molecular signatures.


# Workflow

```text
GEO Transcriptomic Dataset (GSE53987)
            ↓
Differential Expression Analysis (limma)
            ↓
GO / KEGG Functional Enrichment
            ↓
STRING PPI Network Construction
            ↓
Hub Gene Identification (cytoHubba MCC)
            ↓
Network-Constrained Disease Signature
            ↓
Transcriptomic Signature Reversal
            ↓
Potential Repurposed Drug Ranking
            ↓
Validation Using Known Antipsychotics
```

---

# Dataset

- **Source:** GEO (Gene Expression Omnibus)
- **Dataset:** GSE53987
- **Tissue:** Human hippocampus
- **Conditions:** Schizophrenia vs Control
- **Platform:** Affymetrix microarray

---

# Methods

## 1. Differential Expression Analysis

Differential expression analysis was performed using the `limma` package in R.


## 2. Functional Enrichment Analysis

Gene Ontology (GO) and KEGG pathway enrichment analyses were performed to identify dysregulated biological processes and pathways associated with schizophrenia.

---

## 3. Protein–Protein Interaction Network

A PPI network was constructed using STRING and analyzed in Cytoscape. <img width="2925" height="2490" alt="STRING_network" src="https://github.com/user-attachments/assets/5c551195-9b81-448b-b443-a5df7b43ba55" />


Hub gene ranking method: cytoHubba MCC algorithm

Hub genes identified from the network were used to construct a network-constrained schizophrenia transcriptomic signature. <img width="2925" height="2490" alt="STRING network_MCC_top30" src="https://github.com/user-attachments/assets/e86ae642-542a-4085-8301-324c5e935c62" />


---

## 4. Network-Guided Transcriptomic Signature Reversal

Instead of using all differentially expressed genes, transcriptomic reversal analysis was restricted to network-central schizophrenia genes. 

### Directional Reversal Logic

Using the `enrichR` package, we query the **Drug Perturbations from GEO** database:
1. **Reversing Up-regulated Genes:** We take our up-regulated disease network genes and query the `Drug_Perturbations_from_GEO_down` database to find drugs that significantly *down-regulate* these targets.
2. **Reversing Down-regulated Genes:** We take our down-regulated disease network genes and query the `Drug_Perturbations_from_GEO_up` database to find drugs that significantly *up-regulate* these targets.

---

## 5.  Drug Nomenclature Standardization & Consensus Ranking
The outputs from `enrichR` contain highly specific term strings (e.g., "Olanzapine_MCF7_10uM"). To evaluate the drug itself regardless of the cell line or dosage used in the GEO perturbation, we must parse and aggregate the results.

1. **String Extraction:** We use regular expressions to isolate the primary drug name from the complex term string and convert it to lowercase to prevent case-sensitive mismatching.
2. **Score Aggregation:** Since a single drug may appear multiple times (tested across different cell lines or conditions), we group by the drug name and extract the strongest statistical evidence—taking the minimum Adjusted P-value and the maximum Combined Score.
3. **Consensus Intersection:** By performing an `inner_join`, we enforce a strict biological filter: a drug is only considered a valid therapeutic candidate if it successfully reverses *both* the up-regulated and down-regulated components of the schizophrenia network.

## 6.  Scoring & Filtering of Therapeutic Candidates
With our consensus list established, we now refine and rank the drug candidates to extract biologically meaningful therapies.

1.  **Network Reversal Scoring:** We calculate a `network_reversal_score` by taking the minimum (`pmin`) of the Up and Down combined scores. This acts as a bottleneck function, ensuring that a top-ranked drug must robustly reverse *both* directions of the disease signature, rather than just strongly affecting one.
2.  **Toxin & Chemotherapy Exclusion:** Transcriptomic perturbation databases often flag global cellular stressors (like hydrogen peroxide or environmental toxins) and aggressive chemotherapies (like cisplatin or paclitaxel) because they cause massive, widespread gene expression changes. We filter these out to prevent highly toxic false-positives from crowding our final results.
3.  **Final Aggregation:** Finally, we aggregate the mean reversal scores for each drug and sort the list in descending order to present the Top 20 most promising computational drug candidates.

---

# Results / Key Findings

## Biological Themes

Functional enrichment and network-guided transcriptomic analysis revealed strong convergence around mitochondrial dysfunction, oxidative phosphorylation, and proteostasis-related pathways in schizophrenia.

The schizophrenia-associated transcriptomic network showed significant enrichment for:

- Oxidative phosphorylation
- Electron transport chain dysfunction
- Mitochondrial ATP synthesis
- Ubiquitin–proteasome system dysregulation
- Autophagy and mitophagy
- Synaptic vesicle transport
- Neurodegeneration-associated pathways

These findings are consistent with emerging evidence linking schizophrenia to impaired neuronal bioenergetics, oxidative stress, mitochondrial dysfunction, and disrupted synaptic protein homeostasis.

---

## Top Candidate Compounds

| Drug | Biological Relevance |
|---|---|
| Rosiglitazone | PPAR-γ agonist, anti-inflammatory & metabolic regulation |
| Pioglitazone | Immunometabolic modulation |
| Troglitazone | PPAR-γ signaling |
| Resveratrol | Antioxidant & mitochondrial support |
| Estradiol | Neuroendocrine modulation |

---

# Internal Validation

The workflow successfully recovered known antipsychotic compounds including:

- Olanzapine
- Haloperidol

This supports the biological validity of the transcriptomic reversal framework.

---

# External Validation via Connectivity Map (CLUE.io)

While the internal network pharmacology pipeline utilizes mathematical and graphical models (Protein-Protein Interactions) to predict drug efficacy, it is critical to cross-reference these theoretical predictions with empirical biological data. To achieve this, we utilized the **Broad Institute’s Connectivity Map (CMap / CLUE.io)**. 

CMap is a massive perturbation database containing over a million gene expression profiles from human cell lines treated with various FDA-approved drugs, experimental small molecules, and genetic inhibitors. By querying our schizophrenia disease signature against this database, we search for empirical **Transcriptomic Reversal**:

1. **The Query:** We isolated the top 150 significantly up-regulated and down-regulated genes from our disease model to generate a targeted signature.
2. **The Comparison:** CMap systematically compares this disease state against its library of drug-induced states. 
3. **The Connectivity Score (norm_cs):** * A **negative score** indicates that a drug empirically induces a transcriptomic state *opposite* to the disease (e.g., it down-regulates the genes that schizophrenia up-regulates). These are our **Therapeutic Candidates (Reversers)**.
   * A **positive score** indicates that a drug mimics the disease signature, potentially exacerbating the pathology.

By demonstrating that the molecular hubs identified by our *in-silico* network model align tightly with the top-scoring *empirical* reversers in the CMap database, we establish highly robust, dual-validated candidates for drug repurposing.

---

# Final Conclusion & Synthesis: Dual-Validation

The primary objective of this pipeline was to discover novel drug repurposing candidates for schizophrenia by targeting the underlying transcriptomic abnormalities in the hippocampus. By employing two orthogonal approaches—an **internal network-guided structural scoring system** and an **external empirical validation via CLUE.io**—we identified highly concordant therapeutic paradigms. 

The correlation between our computationally derived network scores and the real-world *in vitro* perturbations yields three critical insights:

### 1. The Resveratrol / SIRT1 Axis (A Dual-Validated Target)
The most striking discovery in this analysis is the independent convergence of both pipelines on the **Sirtuin (SIRT) pathway**.
* **Internal Pipeline:** Identified **Resveratrol** as the #2 highest-scoring candidate overall, indicating a massive structural capacity to disrupt the disease network.
* **External Validation:** Independently identified **SIRT Activators** as the #1 most potent mechanistic class for reversing the schizophrenia transcriptomic signature. 
* **Synthesis:** Resveratrol is the canonical SIRT1 activator. The fact that a purely structural network model prioritized this specific molecule, while an empirical transcriptomic database prioritized its exact pharmacological class, provides exceptionally high confidence in SIRT1 activation as a disease-modifying mechanism.

### 2. A Paradigm Shift: Epigenetics & Metabolism
Both analytical methods strongly suggest that reversing the core transcriptomic signature requires **metabolic and epigenetic modulation**, rather than classical neurotransmitter blockade.
* **The Rosiglitazone Validation:** Our internal structural scoring identified **Rosiglitazone** (a PPAR-γ agonist and metabolic modulator) as the absolute highest-scoring candidate in the entire pipeline by a massive margin. 
* **CMap Corroboration:** The CLUE.io empirical data fully validates this shift. The top transcriptomic reversers in CMap were dominated by epigenetic modulators (**HDAC Inhibitors**, **Bromodomain Inhibitors**) and metabolic/stress regulators (**FXR Antagonists**, **JNK Inhibitors**). This provides the necessary biological plausibility for Rosiglitazone’s mathematical dominance in our network model.

### 3. Successful Detection of Standard Therapeutics
To validate the model’s sensitivity, we assessed the performance of known standard-of-care antipsychotics (e.g., **Olanzapine**, **Haloperidol**). Our network successfully detected and captured the transcriptomic signatures of these clinical compounds (yielding reliable baseline scores of 6.71 and 1.55, respectively).

---

# Future Directions
The dual-validation achieved in this pipeline strongly supports the pivot toward neuro-metabolic and epigenetic drug repurposing for Schizophrenia. The top candidates—specifically **Resveratrol (SIRT1 activation)** and **Rosiglitazone (PPARγ agonism)** —warrant immediate downstream investigation.

# Technologies Used

## Programming Language

- R

## R Libraries

- GEOquery
- limma
- dplyr
- ggplot2
- clusterProfiler
- enrichR
- STRINGdb

## External Tools

- Cytoscape
- STRING Database
- cytoHubba
- clue.io

---

# Limitations

This study is based on a single bulk-tissue transcriptomic dataset with limited covariate adjustment and relies on public drug perturbation databases for computational prediction. Consequently, the identified compounds should be interpreted as hypothesis-generating therapeutic candidates rather than experimentally validated treatments for schizophrenia.

# Citation

If you use this repository, please cite the original GEO dataset (https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE53987) and associated software packages.

---

# Author

**Neehar Haldar**
