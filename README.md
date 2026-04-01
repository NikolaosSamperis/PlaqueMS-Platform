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

## Database Architecture

PlaqueMS is supported by a dual-database architecture combining MySQL (relational) and Neo4j (graph), bridging data storage efficiency with biological complexity while enabling efficient querying, relationship-driven analysis, and both exploratory and predictive workflows.

---

### Relational Database (MySQL)

<p align="center">
  <img src="images/MySQL_DB_schema.png" width="60%">
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

---

### Graph Database (Neo4j)

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

## Results

This section summarises the core modules and functionalities of PlaqueMS, following the structure and workflow presented in the thesis.

---

### 3.1.1 Home Page

<p align="center">
  <img src="images/Home_page.png" width="70%">
</p>

The Home page serves as the central entry point of PlaqueMS, introducing the biological context of atherosclerosis and the platform’s role in integrative multi-omics analysis.

It provides immediate access to core modules and presents precomputed visual summaries (e.g. differential expression outputs), allowing users to quickly explore dataset characteristics without requiring prior interaction.

---

### 3.1.2 Proteins Page

<p align="center">
  <img src="images/Protein_annotations_page.png" width="70%">
</p>

The Proteins module focuses on **protein-level metadata and annotation retrieval**.

Users can:
- Query proteins using gene names or UniProt accession IDs  
- Retrieve protein annotations and mappings  
- Explore protein presence across experiments and datasets  

This module acts as a **linking layer between biological entities and experimental data**, enabling consistent navigation across cohorts.

---

### 3.1.3 Differential Expression Analysis

<p align="center">
  <img src="images/Diff_expression_analysis_page.png" width="45%">
  <img src="images/Volcano_plot.png" width="45%">
</p>

This module enables exploration of precomputed differential expression results across cohorts and plaque phenotypes.

Key features include:
- Interactive volcano plots for identifying significantly regulated proteins  
- Access to statistical measures (log fold change, p-values)  
- Dataset-specific filtering  

It supports identification of molecular signatures associated with plaque progression and phenotype differences.

---

### 3.1.4 Protein Networks Page

<p align="center">
  <img src="images/Protein_networks.png" width="70%">
  <img src="images/Protein_network_node_metrics.png" width="45%">
  <img src="images/Protein_network_edge_metrics.png" width="45%">
</p>

The Protein Networks module provides **systems-level exploration of proteomic interactions**.

Users can:
- Visualise protein–protein interaction (PPI) networks  
- Identify functional modules using clustering approaches (e.g. MCL)  
- Analyse node and edge-level network metrics  
- Integrate expression data into network context  

This enables interpretation of plaque biology beyond individual proteins.

---

### 3.1.5 Authentication & Administration

<p align="center">
  <img src="images/Login_page.png" width="45%">
  <img src="images/Register_new_user_page.png" width="45%">
  <img src="images/Admin_dashboard.png" width="60%">
</p>

PlaqueMS implements a secure authentication system with:

- User registration and login  
- Role-based access control (user/admin)  
- Administrative approval workflows  
- User activity tracking  

Access to advanced analytical and clinical modules is restricted to authorised users.

---

### 3.1.6 PlaQuery & Protein Abundance (Restricted Modules)

<p align="center">
  <img src="images/Protein_search_engine.png" width="45%">
  <img src="images/Filters_section.png" width="45%">
  <img src="images/Results_protein_search_engine.png" width="60%">
</p>

PlaQuery provides **advanced, phenotype-driven querying capabilities** across integrated datasets.

Users can filter based on:
- Clinical phenotype (e.g. calcification, symptoms)  
- Tissue region and plaque type  
- Demographics (age, sex)  
- Experimental conditions  

---

#### Protein Abundance Integration

Within PlaQuery, the **Protein Abundance functionality** enables:

- Retrieval of quantitative protein expression values across subjects  
- Cross-cohort comparison of protein levels  
- Export of results (CSV, Excel, TSV)  
- Structured tabular outputs with associated clinical metadata  

This module directly links **query filters → protein-level quantitative data**, making it a core component of downstream analysis and model input generation.

---

### 3.1.7 Predictive Modelling (Calcification & SYNTAX Score)

<p align="center">
  <img src="images/SYNTAX_score_page.png" width="45%">
  <img src="images/SYNTAX_score_prediction_results.png" width="45%">
</p>

The prediction module integrates machine learning models trained on proteomics data.

Users can:
- Upload protein abundance tables (single-sample or cohort-level)  
- Select predefined protein panels (e.g. Core Matrisome)  
- Apply optional preprocessing (e.g. log₂ transformation)  

The system generates:
- Predicted SYNTAX score (continuous output)  
- Classification outputs for calcification status  
- Structured result tables linked to clinical metadata  

This module connects **data exploration with predictive analytics**, enabling translational insights.

---

### 3.2 Model Performance and Validation

<p align="center">
  <img src="images/Calcification_pred_GUHCL_vs_EQ_Mass_mg.png" width="32%">
  <img src="images/Calcification_pred_GUHCL_vs_Score.png" width="32%">
  <img src="images/Calcification_pred_GUHCL_vs_Volume_mm3.png" width="32%">
</p>

Model outputs demonstrate significant correlation with clinical imaging metrics:

- Calcium mass  
- CT-derived scores  
- Plaque volume  

These results indicate that the models capture **continuous variation in calcification burden**, rather than simple binary classification.

---

### 3.3 Clinical Stratification

<p align="center">
  <img src="images/SDS_violin_asymptomatic.png" width="50%">
</p>

The models enable clinically relevant patient stratification:

- Clear separation between symptomatic and asymptomatic groups  
- Higher predicted calcification probabilities in specific patient subsets  
- Statistically significant differences across clinical categories  

This highlights the potential of PlaqueMS as a **decision-support and hypothesis-generation tool**.

---

## 📄 Full Thesis

For full methodology, evaluation, and discussion:

👉 [PlaqueMS Thesis PDF](./Thesis%20project%20Nikolaos%20Samperis.pdf)

