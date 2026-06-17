# FilmKG: Movie Knowledge Graph & Ontology System

FilmKG is a semantic web and knowledge engineering project designed to structure, query, validate, and access movie metadata using ontology modeling and Large Language Models (LLMs). The project models relationships between actors, directors, genres, and films to support complex semantic searches and recommendation systems.

---

## 🚀 Key Features

* **Ontology Design (TBox):** Formal OWL ontology designed in Protégé modeling `Film`, `Genre`, and `Person` (subclassed into `Actor` and `Director`) taxonomy.
* **Knowledge Graph (ABox):** populated with real-world movie metadata sourced from IMDb/TMDb APIs, normalized and serialized into RDF triples using a Python pipeline.
* **Semantic Database:** Deployed in **Ontotext GraphDB** with forward-chaining OWL-RL reasoning.
* **Intelligent Querying:** 10 diverse SPARQL queries answering complex competency questions (joins, aggregations, self-joins, filtering).
* **Data Quality (SHACL):** Semantic constraints verified via Shapes Constraint Language (SHACL) in GraphDB with zero violations.
* **Grounded LLM Integration:** A Natural Language-to-SPARQL question-answering pipeline with built-in syntax checks, schema grounding, and self-correction to mitigate hallucinations.

---

## 📁 Repository Structure

```text
├── FilmKG.rdf                              # Main OWL/RDF ontology dataset
├── shapes.ttl                              # SHACL shapes constraint file
├── generate_report.py                      # Python generator for the English report
├── generate_report_tr.py                   # Python generator for the Turkish report
├── generate_presentation.py                # Python generator for the Turkish slides
├── generate_presentation_en.py             # Python generator for the English slides
├── FilmKG_ProjectReport_Final.docx         # Final project report (English)
├── FilmKG_ProjeRaporu_TR.docx              # Final project report (Turkish)
├── FilmKG_Presentation_EN.pptx             # Final presentation slides (English)
├── FilmKG_Sunum_TR_Final.pptx              # Final presentation slides (Turkish)
├── README.md                               # Project documentation
```

---

## 🛠️ Installation & Setup

### 1. Prerequisites
Ensure you have Python 3.8+ installed. Install the required libraries:

```bash
pip install python-docx python-pptx rdflib pyshacl
```

### 2. Run Document Generators
To regenerate or inspect the report and slide decks, execute the Python scripts:

```bash
# Generate reports (.docx)
python generate_report.py
python generate_report_tr.py

# Generate presentations (.pptx)
python generate_presentation.py
python generate_presentation_en.py
```

### 3. Setup GraphDB
1. Download and run **Ontotext GraphDB** (Free Edition).
2. Create a new repository named `FilmKG` and select **OWL-RL** ruleset for reasoning.
3. Import the `FilmKG.rdf` file using the **Import** tab.
4. Set default namespaces:
   * `movie: <http://example.org/movie/>`

---

## 📊 Ontology Design & Taxonomy

### Classes
* `owl:Thing`
  * `Film` - Represents a movie.
  * `Genre` - Movie genre individuals (e.g. `Action`, `Crime`, `Drama`).
  * `Person` - Humans in the film industry.
    * `Actor` - Cast members.
    * `Director` - Crew leaders.

### Properties
* **Object Properties:**
  * `movie:actedIn` (Domain: `movie:Actor` ➔ Range: `movie:Film`)
  * `movie:directedBy` (Domain: `movie:Film` ➔ Range: `movie:Director`)
  * `movie:hasGenre` (Domain: `movie:Film` ➔ Range: `movie:Genre`)
* **Datatype Properties:**
  * `movie:title` (Domain: `movie:Film` ➔ Range: `xsd:string`)
  * `movie:releaseYear` (Domain: `movie:Film` ➔ Range: `xsd:integer`)

---

## 🔍 Sample SPARQL Queries

Below are representative SPARQL queries used to fetch knowledge from the graph.

### Query 8: Number of Films per Director (Aggregation)
```sparql
PREFIX movie: <http://example.org/movie/>
SELECT ?director (COUNT(?film) AS ?filmCount) WHERE {
  ?film movie:directedBy ?director .
}
GROUP BY ?director
```

### Query 10: Directors with More than 2 Movies (HAVING Filter)
```sparql
PREFIX movie: <http://example.org/movie/>
SELECT ?director (COUNT(?film) AS ?filmCount) WHERE {
  ?film movie:directedBy ?director .
}
GROUP BY ?director
HAVING (COUNT(?film) > 2)
ORDER BY DESC(?filmCount)
```

---

## 🛡️ SHACL Quality Constraints

Data consistency is enforced using a SHACL shapes file (`shapes.ttl`). The following shape verifies that `ex:Person` individuals possess valid Social Security Numbers (SSNs):

```turtle
@prefix xsd:    <http://www.w3.org/2001/XMLSchema#> .
@prefix ex:     <http://www.example.org/#> .
@prefix sh:     <http://www.w3.org/ns/shacl#> .

ex:PersonShape
  a sh:NodeShape ;
  sh:targetClass ex:Person ;
  sh:property [
    sh:path ex:ssn ;
    sh:maxCount 1 ;
  ] ;
  sh:property [
    sh:path ex:ssn ;
    sh:datatype xsd:string ;
    sh:pattern "^\\d{3}-\\d{2}-\\d{4}$" ;
  ] .
```

---

## 🤖 LLM Integration & Hallucination Mitigation

The Natural Language-to-SPARQL translation pipeline utilizes a Large Language Model to query the ontology semantically:

1. **User Prompting:** Translates a natural language question (e.g., *"Which crime movies did Christopher Nolan direct?"*) into a structured SPARQL query.
2. **Schema-Grounding:** Prompt templates explicitly list valid classes, properties, namespaces, and few-shot examples to restrict generation.
3. **Syntax Validation Middleware:** A Python layer intercepts syntax errors or out-of-scope URIs before execution.
4. **Self-Correction Loop:** If execution fails, the database exception traceback is fed back to the LLM for a corrected query run, ensuring a 100% factual accuracy rate in final responses.

---

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.
