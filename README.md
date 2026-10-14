# AI Data Security

A curated, deep-dive guide and list of open-source tools dedicated to **Data Security and Integrity in Artificial Intelligence**. 

In the era of Large Language Models (LLMs) and advanced Machine Learning (ML), the traditional cybersecurity perimeter has shifted. **The data itself is now the primary attack surface.** If malicious actors can poison the training data, manipulate vector embeddings, or bypass data provenance checks, they can fundamentally compromise the AI model's behavior, leading to data exfiltration, biased decision-making, and autonomous system hijacking. This repository explores the critical pillars of securing AI through rigorous data governance.

## 📑 Table of Contents

1. [Data Processing: The Foundation of Secure AI](#-data-processing-the-foundation-of-secure-ai)
   * [Data Cleaning & PII Sanitization](#data-cleaning--pii-sanitization)
   * [Data Verification & Validation](#data-verification--validation)
   * [Data Lineage](#data-lineage)
   * [Data Integrity](#data-integrity)
   * [Data Provenance](#data-provenance)
   * [Data Augmentation](#data-augmentation--synthetic-data)
   * [Data Balancing](#data-balancing--bias-mitigation)
2. [Data Types: Securing the Attack Surface](#-data-types-securing-the-attack-surface)
   * [Structured Data](#structured-data)
   * [Semi-structured Data](#semi-structured-data)
   * [Unstructured Data](#unstructured-data)
3. [Watermarking: Protecting IP and Authenticity](#-watermarking-protecting-ip-and-authenticity)
4. [Retrieval-Augmented Generation (RAG) Security](#-retrieval-augmented-generation-rag-security)
   * [Vector Stores Security](#vector-stores)
   * [Embeddings & Inversion Attacks](#embeddings)

---

## 🛡️ Data Processing: The Foundation of Secure AI

Secure AI starts long before the model training phase. The data processing pipeline is the first line of defense against adversarial machine learning, data poisoning (where attackers subtly modify training data to introduce backdoors), and privacy breaches.

### Data Cleaning & PII Sanitization
LLMs are infamous for memorizing their training data. If raw datasets contain Personally Identifiable Information (PII) or Protected Health Information (PHI), the model can be prompted to regurgitate this sensitive data, leading to severe GDPR/HIPAA violations. Data cleaning in a security context means aggressive anonymization, masking, and redaction.
* **Security Risk:** Data leakage, model memorization attacks, prompt extraction.
* **[microsoft/presidio](https://github.com/microsoft/presidio):** An open-source framework by Microsoft that provides fast identification and anonymization of PII in text and images using customizable NLP models and rule-based logic.
* **[cleanlab/cleanlab](https://github.com/cleanlab/cleanlab):** The standard AI software for finding and fixing label errors in datasets, preventing attackers from injecting poisoned, mislabeled data into the training pipeline.

### Data Verification & Validation
Before data enters a model, it must strictly conform to expected schemas and statistical distributions. Adversaries often attempt to inject out-of-bounds data or malicious payloads (like hidden prompt injections in scraped web text) to corrupt the model's logic.
* **Security Risk:** Data poisoning, buffer overflows in legacy parsers, indirect prompt injections.
* **[great-expectations/great_expectations](https://github.com/great-expectations/great_expectations):** A powerful Python-based framework that allows data teams to define strict, testable "expectations" (rules) for data quality and structure, blocking anomalous or malicious batches from entering the training pipeline.

### Data Lineage
Understanding exactly *how* data flowed from its source to the model. In the event of a security breach or a model generating toxic output, security engineers must trace the anomalous behavior backward through the pipeline to find the exact transformation step or database query that introduced the flaw.
* **Security Risk:** Inability to perform incident response or forensic analysis on AI anomalies.
* **[OpenLineage/OpenLineage](https://github.com/OpenLineage/OpenLineage):** An open standard for metadata and data lineage collection, allowing security teams to visualize the entire lifecycle of a dataset across complex distributed systems (Airflow, Spark, dbt).

### Data Integrity
Ensuring that data has not been tampered with in transit or at rest. If an attacker compromises a storage bucket and silently alters a few megabytes of training data, they can introduce a "sleeper agent" backdoor into the AI model that remains dormant until triggered by a specific keyword.
* **Security Risk:** Man-in-the-Middle (MitM) data modification, stealthy backdoor injection.
* **[iterative/dvc](https://github.com/iterative/dvc):** Data Version Control. DVC operates like Git for large datasets. It uses cryptographic hashing to track every version of a dataset, ensuring that any unauthorized tampering is immediately detected through hash mismatches.

### Data Provenance
While lineage tracks the *flow*, provenance tracks the *origin and ownership*. Where did this data come from? Who created it? Is it legally licensed, or was it scraped from an untrusted, potentially malicious Dark Web forum? Cryptographic provenance prevents supply chain attacks in AI.
* **Security Risk:** AI supply chain compromises, copyright infringement, ingestion of maliciously crafted datasets.
* **[in-toto/in-toto](https://github.com/in-toto/in-toto):** A framework that cryptographically secures the integrity of the software (and data) supply chain, providing verifiable cryptographic signatures for every step a dataset takes from origin to ingestion.

### Data Augmentation & Synthetic Data
When real-world data is too sensitive (e.g., banking records, medical scans) to be exposed to data scientists or third-party cloud GPUs, security teams use data augmentation and synthetic generation. This creates mathematically similar data that maintains privacy and prevents adversarial reconstruction attacks.
* **Security Risk:** Privacy violations during model training, re-identification attacks.
* **[sdv-dev/SDV](https://github.com/sdv-dev/SDV):** The Synthetic Data Vault. An ecosystem of ML-based models used to generate secure, synthetic tabular, relational, and time-series data that mimics production data without exposing actual PII.

### Data Balancing & Bias Mitigation
In security, bias is a vulnerability. An imbalanced dataset can cause an AI security tool (like a malware classifier or facial recognition system) to systematically fail against specific demographics or novel attack vectors, creating blind spots that adversaries can exploit.
* **Security Risk:** Algorithmic discrimination, systemic classification blind spots.
* **[scikit-learn-contrib/imbalanced-learn](https://github.com/scikit-learn-contrib/imbalanced-learn):** Provides advanced sampling techniques (like SMOTE) to balance datasets, ensuring the model's decision boundaries remain robust and are not skewed by heavily represented, benign data classes.

---

## 🗄️ Data Types: Securing the Attack Surface

The structure of the data dictates the security controls required. Different formats present unique attack vectors for AI systems.

### Structured Data
Highly organized data residing in relational databases (SQL, CSVs). 
* **Security Context:** Easiest to secure using traditional Role-Based Access Control (RBAC), Column-Level Encryption, and tokenization. However, AI models trained on structured data are highly susceptible to **Membership Inference Attacks**, where an attacker queries the model repeatedly to deduce if a specific individual's record was part of the training set.

### Semi-structured Data
Data with some organizational properties but lacking a rigid schema (JSON, XML, server logs).
* **Security Context:** Often contains deeply nested hierarchical data. Attackers can hide malicious payloads, zero-day exploits, or indirect prompt injections deep within JSON values that bypass superficial WAF (Web Application Firewall) checks but are fully processed and executed when ingested by an LLM parsing the logs.

### Unstructured Data
Text documents, emails, PDFs, audio, and images. The primary fuel for modern GenAI and LLMs.
* **Security Context:** The hardest to secure. Unstructured data is a massive attack vector for **Steganography** (hiding malicious code inside image pixels) and **Indirect Prompt Injections** (e.g., an invisible, white-font text hidden on a webpage that commands an AI reading the page to exfiltrate the user's chat history).
* **[Unstructured-IO/unstructured](https://github.com/Unstructured-IO/unstructured):** Open-source libraries for securely partitioning and standardizing complex unstructured data (PDFs, HTML) into clean text ready for LLM consumption, stripping out potentially malicious formatting.

---

## 🌊 Watermarking: Protecting IP and Authenticity

As AI becomes capable of generating photorealistic deepfakes and mass-producing hyper-targeted phishing text, distinguishing between human and AI-generated data is a critical security imperative.

* **Security Context:** Watermarking involves embedding imperceptible, cryptographic statistical signals into the AI's output (either modifying the token selection probability in text or tweaking pixel values in images). This allows organizations to track the origin of disinformation campaigns, prevent copyright theft, and stop the "Model Collapse" phenomenon (where AIs accidentally train on AI-generated garbage data).
* **[jwkirchenbauer/lm-watermarking](https://github.com/jwkirchenbauer/lm-watermarking):** A reference implementation for watermarking the outputs of Large Language Models, demonstrating how to cryptographically embed tracking signals into generated text without degrading the quality of the output.

---

## 🧩 Retrieval-Augmented Generation (RAG) Security

RAG architecture connects a frozen LLM to a live database of external knowledge. While this prevents hallucinations, it introduces a massive new security paradigm: the AI is only as secure as the data it retrieves.

### Vector Stores
Vector databases store the "memories" (embeddings) that the LLM queries.
* **Security Context:** If an attacker gains access to the Vector DB, they can perform **Knowledge Base Poisoning**. By injecting malicious documents into the database, they guarantee that the LLM will retrieve and present false, malicious, or brand-damaging information to the end-user. Vector DBs require rigorous tenant isolation, Row-Level Security (RLS), and encrypted storage.
* **[milvus-io/milvus](https://github.com/milvus-io/milvus) & [qdrant/qdrant](https://github.com/qdrant/qdrant):** Leading open-source vector databases that are actively implementing advanced RBAC, TLS encryption, and secure tenant isolation protocols specifically tailored for RAG security.

### Embeddings
Embeddings are the mathematical representations (high-dimensional vectors) of text, images, or audio stored in the Vector DB.
* **Security Context:** A common misconception is that embeddings are inherently secure hashes. They are not. Through **Embedding Inversion Attacks**, adversaries who steal vector data can train a shadow model to reverse-engineer the original plaintext. This means if a company converts highly confidential legal contracts into embeddings, a breach of those numbers equates to a breach of the actual text. Embeddings of sensitive data must be treated as highly classified intellectual property and encrypted at rest.
