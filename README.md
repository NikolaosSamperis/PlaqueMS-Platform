# PlaqueMS

[![Python 3.11](https://img.shields.io/badge/python-3.11-blue.svg)](https://www.python.org/downloads/release/python-3110/)
[![Django 5.1](https://img.shields.io/badge/django-5.1-brightgreen.svg)](https://docs.djangoproject.com/en/5.1/)
[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)

## *An Integrative Web Platform for Atherosclerosis Omics Analysis.*
Facilitating Visual and Predictive Insights into Atherosclerotic Plaque Biology

## **Project Description**
**PlaqueMS** is an integrative web platform for the exploratory and predictive analysis of human atherosclerotic plaques using multi-omics data, with a primary focus on proteomics. Developed in Django, it provides interactive tools for protein abundance analysis, exploration of differential expression analysis results, protein–protein interaction network visualization (powered by Cytoscape), and predictive modeling of clinically relevant outcomes such as plaque calcification status and SYNTAX score.

The platform integrates a MySQL relational database for structured storage of differential expression results, precomputed visual outputs (e.g. box plots, volcano plots, heatmaps), and protein network-related files. A complementary Neo4j graph database is employed for capturing more complex relationships among patients, clinical metadata, experimental protocols, tissue regions, and protein abundance measurements. This dual-database architecture enables dynamic, phenotype-specific queries across plaque regions and cohorts. Secure user authentication, role-based access control, and an administrative interface support responsible data management and user oversight.

By combining rich exploratory capabilities with machine learning-based prediction, PlaqueMS offers a comprehensive framework for advancing translational cardiovascular research.

<h3 align="center">System Architecture</h3>

<p align="center">
  <img src="images/PlaqueMS_pipeline_diagram_horizontal.drawio.png" alt="PlaqueMS Architecture" width="100%">
</p>

>*High-level overview of the system architecture of PlaqueMS. Solid arrows indicate data flow from the frontend to the backend (user actions, HTTP/AJAX requests, and queries to the database), while dashed arrows represent data or results returned from the backend to the frontend, including rendered web pages, interactive visualizations, and downloadable files. Colored boxes distinguish the platform’s major components: purple denotes frontend user interface modules, green highlights backend logic and Django views, red indicates databases (MySQL and Neo4j), and grey represents external tools and machine learning modules. Certain modules and functionalities are accessible only to authenticated users or administrators. This diagram illustrates the coordinated interactions that enable seamless data analysis and visualization within the platform.*

---
<br>

## Motivation & Research Gaps

Atherosclerosis remains poorly understood at the molecular level due to limitations in current data integration and analytical tools.

### Key Gaps

- **Fragmented multi-omics data**  
  Omics datasets are often analysed in isolation, limiting systems-level insight.

- **Underutilisation of proteomics**  
  Tissue-based proteomics in human plaques is still limited compared to transcriptomics.

- **Small cohorts & limited generalisability**  
  Many studies rely on small datasets or non-human models.

- **Lack of disease-specific tools**  
  Existing platforms are not tailored to atherosclerosis and lack clinical integration.

- **Static data exploration**  
  Most tools provide fixed outputs with limited support for dynamic querying.

- **Limited practical use of machine learning**  
  ML models are rarely integrated into interactive, user-friendly platforms.

### PlaqueMS Contribution

PlaqueMS addresses these gaps by providing:

- Integrated **proteomics + clinical + network data**
- **Phenotype-driven, interactive exploration**
- Hybrid **MySQL + Neo4j architecture**
- Embedded **machine learning predictions**
- A **web-based, user-friendly platform**

This enables dynamic data interrogation and supports more translational cardiovascular research.

---
<br>

## Database Architecture

PlaqueMS is supported by a dual-database architecture combining MySQL (relational) and Neo4j (graph), bridging data storage efficiency with biological complexity while enabling efficient querying, relationship-driven analysis, and both exploratory and predictive workflows.

### 1. Relational Database (MySQL)

<p align="center">
  <img src="images/MySQL_DB_schema.png" width="50%">
</p>

The MySQL database is responsible for the structured storage of core entities, analytical results, and system-level data.

#### Key Components

- **Proteins Table**
  - Stores protein identifiers (UniProt accession IDs, gene names)
  - Serves as a central reference for mapping across datasets

- **Datasets & Experiments**
  - `datasets`: Represents individual cohorts (e.g. Vienna, Athero-Express, Virginia)
  - `experiments_types`:
    - Organises experiments hierarchically as **cohort → protein extraction protocol → comparison/experiment**, with each node linked to a file path  
    - Powers the tree-based interface and allows dynamic retrieval of the relevant results for the selected branch  

- **Differential Expression Results**
  - `diff_result`: Stores statistical outputs (e.g. filenames, filepaths of analysis results such as volcano plots)

- **Protein Networks**
  - `networks`: Contains metadata and file references for protein–protein interaction networks
  - `network_and_experiment`: Links networks to specific experiments

- **Documents & Precomputed Outputs**
  - `statistics`: Stores generated visualisations (heatmaps, boxplots, etc.)
  - `doc_and_experiment`: Associates analytical outputs with experiments

- **User Management**
  - `login_customuser`: Handles authentication, access control, and user roles (admin, approved users, etc.)

#### Key Characteristics

- Uses **UUID-based identifiers** to ensure data integrity and consistent referencing  
- Implements **foreign key relationships** to enforce schema consistency  
- Optimised for **fast retrieval of structured data and precomputed analytical results**  
- Stores **file paths** rather than raw data, enabling efficient file-based access  
<br>

### 2. Graph Database (Neo4j)

<p align="center">
  <img src="images/Neo4j_DB_schema.png" width="40%">
</p>

The Neo4j database models complex biological and clinical relationships that are inefficient to represent and traverse using a relational (table-based) schema.

#### Core Node Types

- **Patient**
  - Represents individual subjects with associated clinical metadata

- **Experiment**
  - Encodes experimental context (cohort, protein extraction protocol, plaque region)

- **Protein**
  - Represents quantified proteins across samples

- **Core / Periphery**
  - Captures spatial plaque regions (e.g. core vs periphery)

#### Relationships

- Patients are linked to experiments  
- Experiments are linked to proteins via quantified protein abundance measurements  
- Proteins are connected via inferred protein–protein interactions derived from conditional mutual information–based network analysis  
- Experiments are associated with spatial plaque regions (e.g. core, periphery)    

#### Key Characteristics

- Enables phenotype-driven queries by integrating patient-level clinical metadata, plaque regions, and experimental context to explore protein abundance across cohorts  
- Supports multi-hop traversal (e.g. patient → experiment → region → protein), enabling integrative analysis across data layers  
- Allows dynamic, flexible querying without complex joins required in relational databases  
- Complements MySQL by modelling and querying highly connected biological relationships  

---
<br>

## Results

This section summarises the core modules and functionalities of PlaqueMS, following the structure and workflow presented in the thesis.
<br>

### 1. Home Page

<p align="center">
  <img src="images/Home_page.png" width="70%">
</p>

The Home page serves as the entry point of PlaqueMS, introducing the biological context of atherosclerosis and the purpose of the platform, and directing users to the available analytical modules.
<br>
<br>

### 2. Proteins Module

<p align="center">
  <img src="images/Protein_annotations_page.png" width="70%">
</p>

The Proteins module provides access to protein identifiers and annotation information.

Users can:
- Query proteins using gene names or UniProt accession IDs  
- Retrieve corresponding Uniprot accession IDs, UniProtKB IDs and gene name mappings  

This module supports consistent identification and mapping of proteins across datasets.
<br>
<br>

### 3. Differential Expression Analysis Module

<p align="center">
  <img src="images/Diff_expression_analysis_page.png" width="49%">
  <img src="images/Volcano_plot.png" width="49%">
</p>

This module provides access to precomputed differential expression results across cohorts and experimental comparisons.

Key features include:
- Visualisation of results (e.g. volcano plots, boxplots)  
- Access to statistical measures (log fold change, p-values)  
- Filtering by dataset, protein extraction protocol, plaque region, and experimental comparison via the hierarchical structure  

It supports comparison of protein-level differences across conditions and plaque regions.
<br>
<br>

### 4. Protein Networks Module

<p align="center">
  <img src="images/Protein_networks.png" width="70%">
  <img src="images/Protein_network_node_metrics.png" width="49%">
  <img src="images/Protein_network_edge_metrics.png" width="49%">
</p>

The Protein Networks module enables visualisation and exploration of protein–protein interaction networks derived from proteomic data using Cytoscape.

Users can:
- Visualise network structure  
- Apply clustering methods (e.g. Markov Clustering)  
- Inspect node and edge attributes (e.g. mutual information, p-values, log fold change)  
- Overlay differential expression results onto the network
- Export node and edge data, as well as the generated network visualisation (PNG), for downstream analysis  

This supports exploration of protein interactions within the context of the analysed datasets.
<br>
<br>

### 5. Authentication & Administration

<p align="center">
  <img src="images/Login_page.png" width="49%">
  <img src="images/Register_new_user_page.png" width="49%">
  <img src="images/Admin_dashboard.png" width="60%">
</p>

PlaqueMS implements a secure authentication and user management system, including:

- User registration and login  
- Role-based access control (user/admin)  
- Administrative approval of new user accounts  

Access to selected modules and functionalities is restricted to authorised users.
<br>
<br>

### 6. PlaQuery & Protein Abundance (Restricted Modules)

<p align="center">
  <img src="images/Protein_search_engine.png" width="49%">
  <img src="images/Filters_section.png" width="49%">
  <img src="images/Results_protein_search_engine.png" width="60%">
</p>

PlaQuery enables filtering and retrieval of protein abundance data across integrated cohorts using clinical, demographic, and experimental variables.

Users can:
- Search proteins using gene names or identifiers  
- Apply filters based on clinical phenotype (e.g. calcification, symptoms), tissue region, demographics (age, sex), and experimental context  
- Retrieve quantitative protein abundance values at the patient level  
- View results in structured tabular format with associated metadata (e.g. patient ID, tissue area, experiment)  
- Export results (CSV, Excel, TSV)  

This module supports exploration of protein abundance across cohorts and conditions through flexible, filter-based queries.
<br>
<br>

### 7. Predictive Modelling (Calcification & SYNTAX Score Modules)

<p align="center">
  <img src="images/SYNTAX_score_page.png" width="49%">
  <img src="images/SYNTAX_score_prediction_results.png" width="49%">
</p>

PlaqueMS includes two separate predictive modelling modules based on machine learning models trained on proteomics data:

- **Calcification Prediction Module**
  - Predicts plaque calcification status (classification with associated probabilities)

- **SYNTAX Score Prediction Module**
  - Predicts SYNTAX score (continuous output)

Users can:
- Upload protein abundance data (single sample or cohort-level)  
- Select predefined protein subsets (e.g. Core Matrisome)  
- Apply optional preprocessing (e.g. log₂ transformation)  

These modules enable application of trained models to new proteomic data and can support estimation of calcification status or SYNTAX score in similar cohorts lacking these clinical measurements.
<br>
<br>

### 8. Model Performance and Validation

<p align="center">
  <img src="images/Calcification_pred_GUHCL_vs_EQ_Mass_mg.png" width="32%">
  <img src="images/Calcification_pred_GUHCL_vs_Score.png" width="32%">
  <img src="images/Calcification_pred_GUHCL_vs_Volume_mm3.png" width="32%">
</p>

Model predictions of calcification probability show moderate positive correlations with clinical imaging-derived measures, including:

- Calcium mass  
- Agatston score  
- Plaque volume  

Correlations were assessed using Spearman’s rank correlation coefficient, indicating consistent associations between predicted probabilities and quantitative measures of calcification burden.

These results suggest that, for the evaluated model, predicted probabilities reflect gradation in calcification burden across samples, rather than representing purely binary classification outcomes. This is useful as it enables differentiation between varying levels of calcification severity, supporting more nuanced interpretation of disease state across samples.
<br>
<br>

### 9. Clinical Stratification

<p align="center">
  <img src="images/SDS_violin_asymptomatic.png" width="50%">
</p>

Predicted calcification probabilities differ between symptomatic and asymptomatic patients, with asymptomatic patients showing slightly higher median values.

Statistical testing indicates that this difference is unlikely to be due to chance (p = 0.0412), although there is considerable overlap between the two groups and only modest separation (AUC = 0.63).

As this was evaluated using cross-cohort validation, the results suggest that the model captures some relationship between predicted calcification and symptom status in an independent dataset, but the effect is limited.

---
<br>

## 📄 Full Thesis

For full details on methodology, results, and evaluation, refer to the complete thesis:

👉 [PlaqueMS Thesis PDF](./Thesis%20project%20Nikolaos%20Samperis.pdf)

---
<br>

## 📄 License

This project is licensed under the **Apache License 2.0**.  

Copyright © 2026 Nikolaos Samperis

See the [LICENSE](LICENSE) file for full details.

---

