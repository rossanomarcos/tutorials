

# tellic knowledgen Library vs 1.0

## Scope
> The goal of this document is to provide an overview of the tellic knowledgen library, a
> specification for the interfaces for exposed methods, installation instructions, and a
> usage guide.

## Overview
> The tellic Knowledge library is a python package (also named tellic_nlp) for performing
> various NLP tasks in the biomedical domain. On structured data (tables or triples), it can
> be used to normalize user-specified columns to specific ontologies. On unstructured text
> data (documents), it can be used for named entity recognition, automated ontology
> mapping, sentence segmentation, biomedical content detection, and relationship
> extraction.


## Modules

**Biomedical Content Detection and Filtering**
> The biomedical content detection and filtering module produces a biomedical content score (between 0 and 1) for a free-text document. 
> The score is appended to the result object. If the score exceed a threshold setting, the document is deemed relevant and passed on to downstream processing.

**Named Entity Recognition**
> The named entity recognition module reads in a free-text document, identifies all the named biomedical entities and their corresponding start and end character indices, and appends them to the result object.
> NER is currently supported for the following entity types:
- `Disease`
- `Phenotype`
- `Gene`
- `Variant`
- `Drug`
- `CellType`

**Automated Ontology Mapping**
> The automated ontology mapping module takes the extracted entities and, for each detected entity mention, 
> maps it to a corresponding ontology concept, in the default or user-defined ontology for that entity type. 
> These results are appended to the result object.
> AOM is currently supported for the following entity-ontology pairs:
- `"Disease"` -> `"MESH"`
- `"Phenotype"` -> `"EFO"`
- `"Gene"` -> `"HGNC"`
- `"Variant"` -> `"tellic-VO"` (tellic-internal Variant Ontology)
- `"Drug"` -> `"ChEMBL"`
- `"CellType"` -> `"Cell Ontology"`

**Triple Normalization**
> The triple normalization module takes in rows (represented as JSON dictionaries) from a structured (columnar) source and uses the AOM module to map the items from user- specified columns to a specified target ontology. 
> The results are returned in the same format with the mapped concept ids and concept preferred names added to the JSON dictionary.

**Install Requirements**
- `40 GB of disk space`
- `Python 3.6`

**Python Package Dependencies**
- `spacy==2.2.4`
- `scikit-learn==0.21.3`
- `fasttext`
- `flashtext`
- `glibc`
- `lightgbm`
- `spacy-transformers==0.5.0`
- `unidecode`
- `BeautifulSoup4`
- `lxml`
- `python-dateutil`

**Install Instructions**
- `1. Copy tellic-nlp.tar.gz to machine`
- `2. Run pip install tellic-nlp.tar.gz in python 3.6 environment`
- `3. Confirm installation by starting a python shell and running:`
    -  `import tellic_nlp`


Usage Guide
===============

# The DocumentCurator Class
> This class contains all the methods for document level processing.

```python
Class DocumentCurator(configs=None):
    def process_document(document)
    def extract_entities(document, entity_type)
    def segment_sentences(document)
    def detect_biomedical_content(document)
 ```

**Configuration**
> Before use, a ***DocumentCurator*** object needs to be instantiated with the desired
> processing ***configuration***, ***supplied*** as a ***dictionary***. 
> The format is defined as follows:

````json
{
  “biomedical_content_flag”: bool,
  “biomedical_content_threshold”: float,
  “entity_extraction_flag”: bool,
  “entity_type_list”: List(dict)
  [
     {
       “ontology”: str
       “entity_type”: str,
     },
     {
       “entity_type”: str,
       “ontology”: str
     },
     ...
    ],
    “relationship_flag”: bool,
    “relationship_type_list”: List(dict)
    [
    {
    “entity_1_type”: str,
    “entity_2_type”: str
    },
    {
    “entity_1_type”: str,
    “entity_2_type”: str
    },
    ...
  ]
}
````
**Option Definitions**
- `"biomedical_content_flag"` – whether or not to run biomedical content scoring. If not included, defaults to True
- `"biomedical_content_threshold"`- value between 0 and 1, if score below threshold skip downstream processing. If not included, defaults to 0 (no filtering)
- `"entity_extraction_flag"` – whether or not to run entity extraction and normalization. If not included, defaults to True
- `"entity_type_list"` - list of dictionaries that defines the entity types to extract and ontologies to map to. 
> Currently, supported ***entity*** types include:
>> - `“Disease”` 
>> - `“Phenotype”` 
>> - `“Gene”` 
>> - `“Variant”` 
>> - `“Drug”` 
>> -  and `“CellType”`

> Currently, supported ***ontologies*** are:
>> - `“MESH”` for Disease
>> - `“EFO”` for Phenotype
>> - `“HGNC”` for Gene
>> - `“tellic-VO”` for Variant
>> - `“Cell Ontology”` for CellType
>>     If not included, defaults to all currently supported ***entiy/ontology*** pairs.
    
- `"relationship_flag"` – whether or not to run relationship extraction. If not included, defaults to True
- `"relationship_type_list"` – list of dictionaries that defines the relationship types to extract.
    Defined as pairs of entity types (potentially both the same type). If not included, defaults to all pairs.

**Methods**
- `"process_document(document)"` – runs the full suite of NLP functions over a document and returns the results as a dictionary. Full output format defined below.
- `"extract_entities(document, entity_type)"` – extracts entity mentions of specific type from document text and returns a list of tuples: `"(mention, span_start, span_end, entity_type, concept_id)"` where:
    - `"mention"` is the text mention of the extracted entity
    - `"span_start"` is the character start position of the entity span
    - `"span_end"` is the character end position of the entity span
    - `"entity_type"` is the type of the entity
    - `"concept_id"` is the mapped concept id in the ontology for the entity type -1 if not found.
- `"segment_sentences(document)"` – splits document text into sentences. Returns a list of tuples of form: `"(sentence_no, start_ix, end_ix, text)"` where:
    - `"sentence_no"` is the sentence number in the document 0-indexed
    - `"start_ix"` is the character start position of the sentence
    - `"end_ix is"` the character end position of the sentence 
    - `"text"` is the sentence text
- `"detect_biomedical_content(document)"` – scores document text for biomedical relevance. Returns a float between 0 and 1

**Output Format**
> The result object for full text curation is structured as follows:
    
````json
{
  “classification”: List(dict)
    [{“type”: str, “score”: float, “flag”: bool}, ...],
  “entity_extraction”: dict
    {
      “entity_type_list”: List(dict)
      [{“entity_type”: str, “ontology”: str}, … ],
      “normalized_entity_list”: List(dict)
        [{“mention”: str, “span_start”: int, “span_end”: int, “entity_type”: str,
        “concept_id”: str, “entity_no”: int}, …]
    },
   “sentence_list”: List(dict)
      [{“sentence_no”: int, “start_ix”: int, “end_ix”: int, “text”: str}, …],
   “relationship_list”: List(dict)
      [{“sentence_no”: int, “entity_1_no”: int, “entity_2_no”: int, “score”: float}]
}
````
**Output Field Definitions**
- `"classification"` – contains the results of biomedical content detection, which future extensibility to other classification modules
- `"type"` – the module type (e.g. BIOMEDICAL_CONTENT)
- `"score"` – biomedical content score between 0 and 1
- `"flag"` – if threshold is set, true if score >= threshold, otherwise false
- `"entity_extraction"` – contains the results of entity extraction and normalization
- `"entity_type_list"` – the entity_type and ontology list defined by the user
- `"normalized_entity_list"` – list of normalized entities, along with an entity_no for the relationship module to reference
- `"sentence_list"` – list of sentences, with sentence_no for relationship module to reference
- `"relationship_list"` – list of candidate relationships and strength score
- `"sentence_no"` – the sentence in the document for the candidate relationship
- `"entity_1_no"` – the first entity in the document for the candidate relationship
- `"entity_2_no"` – the second entity in the document for the candidate relationship
- `"score"` – the strength score for the candidate relationship between 0 and 1



# The TripleCurator Class
> This class contains all the methods related to triple curation.
    
````python
class TripeCurator(configs=None):
    def process_tripe(tripe)
    def normalize_term(text, entity_type, target_ontology)
````

**Configuration**
> Before use, a ***TripleCurator*** object needs to be instantiated with the desired processing
> ***configuration***, ***supplied*** as a ***dictionary***. The format is defined as follows:
`````json
{
  “entity_type_list”: List(dict)
    [
     {
       “entity_type”: str,
       “ontology”: str
     },
     {
      “entity_type”: str,
      “ontology”: str
     },
     ...
   ],
    “column_list”: List(dict)
   [
    {
      “column_name”: str,
      “entity_type”: str
    },
    {
      “column_name”: str,
      “entity_type”: str
    },
    ...
  ],
}
```````

**Example from ***Abbvie*** batch script:**
`````python
    fdb_sm_gene_config= {
        "entity_type_list": [
            {
                "entity_type": "Gene",
                "ontology": "HGNC"
            }
        ],
        "column_list": [
            {
                "column_name": "entity2",
                "entity_type": "Gene"
            }
        ]
    }
`````

**Option Definitions**
- `"entity_type_list"` – list of dictionaries that defines the ***entity*** types to that will be processed and ***ontologies*** to map to.
> Currently, supported ***entity*** types include:
>> - `"Disease"`
>> - `"Phenotype"` 
>> - `"Gene"`
>> - `"Variant"` 
>> - `"Drug"`
>> - and `"CellType"`

> Currently, supported ***ontologies*** are:
>> - `"MESH"` for `"Disease"`
>> - `"EFO"` for `"Phenotype"`
>> - `"HGNC"` for `"Gene"`
>> - `"tellic-VO"` for `"Variant"`
>> - `"Cell Ontology"` for `"CellType"`
>> If not included, defaults to all currently supported entity/ontology pairs.
   
- `"column_list"` – list of dictionaries that defines the columns to be normalized and the entities that they should be mapped to. 
                    The ontologies used will be those defined in the entity type list.

**Methods**
- `process_triple(triple)` – Takes a triple in json format and normalizes specified columns to specified entity types. 
                               Returns the transformed row in json format.
- `normalize_term(text, entity_type)` – maps a raw text mention to the concept id for a specified entity type. 
                                The ontology used will be the one defined during TripleCurator instantiation. Returns the concept id as a string

**Example ***TripleCurator*** configuration / INPUTs**
 `````json
> {
>     "entity_type_list": [
>       {"entity_type": "Gene", "ontology": "HGNC"},
>       {"entity_type": "Disease", "ontology": "MESH"},
>       {"entity_type": "Phenotype", "ontology": "EFO"},
>       {"entity_type": "CellType", "ontology": "Cell Ontology"},
>       {"entity_type": "Drug", "ontology": "ChEMBL"},
>       {"entity_type": "Variant", "ontology": "tellic-VO"}
>     ],
>     "column_list": [
>       {"column_name": "intervention", "enitty_type": "Drug"},
>       {"column_name": "condition", "entity_type": "Disease"}
>     ]
> }
`````

**Example ***structured*** data INPUTs**
`````python
input_row_1 = {
    "nct_id": "NCT01438424",
    "intervention": "Lamivudine",
    "condition": "Hepatitie B Virus"
}
input_row_2 = {
    "nct_id": "NCT00134043",
    "intervention": "Vorinostat",
    "condition": "Thyroid Cancer"
}
input_row_3 = {
    "nct_id": "NCT02946544",
    "intervention": "Trazodone",
    "condition": "Sleep Apnea"
}
`````

**Example ***TripleCurator*** instantiation and usage**
`````python
ct_curator = TripleCurator(configs)

output_row_1 = ct_curator.process_triple(input_row_1)
output_row_2 = ct_curator.process_triple(input_row_2)
output_row_3 = ct_curator.process_triple(input_row_3)               

`````                                                  


**Example ***TripleCurator*** from Abbvie**
`````python

global curator
curator= TripleCurator(configs=config)
output_triple = curator.process_triple(triple_string)

`````

**Example ***TripleCurator*** process triple OUTPUT**
`````python
{
    "nct_id": "NCT01438424",
    "intervention": "Lamivudine",
    "intervention_concept_id": "CHEMBL141",
    "condition": "Hepatitie B Virus",
    "condition_concept_id": "MESHD006509"
}
{
    "nct_id": "NCT00134043",
    "intervention": "Vorinostat",
    "intervention_concept_id": "CHEMBL98",
    "condition": "Thyroid Cancer",
    "condition_concept_id": "MESHD013964"
}
{
    "nct_id": "NCT02946544",
    "intervention": "Trazodone",
    "intervention_concept_id": "CHEMBL621",
    "condition": "Sleep Apnea",
    "condition_concept_id": "MESHD012891"
}
`````