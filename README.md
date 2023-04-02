# Coner Document Viewer (CDA)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)
[![GitHub release](https://img.shields.io/github/release/vliegenthart/coner_document_analyser.svg)](https://github.com/vliegenthart/coner_document_analyser/releases)

## WARNING: This repository is no longer maintained

CDA generates a candidate set of publications to receive feedback on in CIDV, by analysing publications from the corpus and rank them according to features like number of times paper has been cited, number of distinct automatically extracted entities for each category and availability of PDF.

## Coner Collaborative NER Pipeline
Named Entity Recognition (NER) for rare long-tail entities as e.g., often found in domain-specific scientific publications is a challenging task, as typically the extensive training data and test data for fine-tuning NER algorithms is lacking. Recent approaches presented promising solutions relying on training NER algorithms in a iterative distantly-supervised fashion, thus limiting human interaction to only providing a small set of seed terms. Such approaches heavily rely on heuristics in order to cope with the limited training data size. As these heuristics are prone to failure, the overall achievable performance is limited. In this paper, we therefore introduce a collaborative approach which incrementally incorporates human feedback on the relevance of extracted entities into the training cycle of such iterative NER algorithms. This approach, called Coner, allows to still train new domain specific rare long-tail NER extractors with low costs, but with ever increasing performance while the algorithm is actively used. 

Coner consists of 3 modules, namely the Coner Document Analyser (CDA), [Coner Interactive Viewer (CIVD)](https://github.com/vliegenthart/coner_interactive_viewer) and [Coner Feedback Analyser (CFA)](https://github.com/vliegenthart/coner_feedback_analyser). Module 1 (CDA) generates a candidate set of publications to receive feedback on in CIDV, by analysing publications from the corpus and rank them according to features like number of times paper has been cited, number of distinct automatically extracted entities for each category and availability of PDF. Module 2 (CIDV) visualizes the extracted entities and allows users to both give feedback on categorical relevance and select new relevant entities. Finally, module 3 (CFA) analyses the given feedback and generates a list of entities for each category with metadata like relevance score, amount of feedback, etc. This final output can be utilized by the NER algorithm to improve the expansion and filtering step of it's training cycle for the next iteration.

You can read more about CDA and the Coner pipeline in the [Coner Collaborative NER paper](https://github.com/vliegenthart/coner_interactive_viewer/blob/master/public/pdf/coner.pdf).

## Dependencies
CDA's pip package dependencies can be found in `requirements.txt`. Furthermore CDA expects the [PDFNLT Repository](https://github.com/KMCS-NII/PDFNLT-1.0) to be located in the parent directory. [Arxiv.py](https://github.com/lukasschwab/arxiv.py) is used to fetch publication PDFs from Arxiv and [scholar.py](https://github.com/ckreibich/scholar.py) returns meta-data about a publication, including the number of times it has been cited.

## Run CDA module
The following steps are needed to generate entity annotations for desired PDF papers with CDA: 
- Copy `demo-config.py` to `config.py` and insert root paths for your project folders
- fetch_publications.py:
  - Parameters: name of database data folder e.g. `tse_ner`, number of papers to fetch from database, number of items to skip from returned collection, versioning number used to name overview csv for databases
  - Description: Fetches papers from conferences as indicated in the 'booktitles' array; meta-data from server, various download methods for PDF, full text and number of citations retrieval
  - Example script execution: `python fetch_publications.py tse_ner 20000 0 1`
  - Booktitles usage should be set in the script itself

2 Options:
- Copy data/top_full_text/ to [TSE-NER](https://github.com/mvallet91/SmartPub-TSENER)
- Copy paper_overview_total.csv to TSE-NER conference directories
- Extract entities from data/top_full_text/ in TSE-NER
- Replace `{conference}_papers_overview_total` here with new one generated in TSE-NER
- Copy and replace new entity sets from TSE-NER if performance better

OR 

- Extract entities with TSE-NER for paper corpus for both facets

- generate_overview_top_by_citations_and_has_pdf.py:
  - Parameters: name of database data folder e.g. `tse_ner`
  - Description: Generates a new overview of fetched papers for conferences filtered on presence of pdf file and sorted on number of citations per paper BUT also limited on seedsize and entities resulted from TSE-NER iteration number (seedsize and iteration number in `config.py`). Also generates list of most interesting papers for viewer based on filters and sorting and copies PDFs for top X papers of interesting conference to `data/viewer_pdfs/`
  - Only uses viewer booktitles from `config.py`
  - Example script execution: `pgenerate_overview_top_by_citations_and_has_pdf.py tse_ner` 

- generate_overview_has_pdf_and_entities.py:
  - Parameters: name of database data folder e.g. `tse_ner`
  - Description: Generates a new overview of fetched papers for conferences filtered on presence of pdf file and sorted on number of citations per paper. Also generates list of most interesting papers for viewer based on filters and sorting and copies PDFs for top X papers of interesting conference to `data/viewer_pdfs/`
  - Only uses viewer booktitles from `config.py`
  - Example script execution: `generate_overview_has_pdf_and_entities.py tse_ner` 

- pdfnlt_find_occurrences.sh:
  - Prerequisites: 
    - Have [PDFNLT](https://github.com/KMCS-NII/PDFNLT-1.0) directory in same parent directory as this project
    - Put PDFs to be analysed in `../PDFNLT/pdfanalyzer/pdf/` directory
    - Each PDF needs entity set files (simple text files where every line contains 1 entity) for each facet with file name location `/data/<database_name>/<conference_name>/entity_set/<facet>_<publication_id>_entity_set_0.txt`
    - Have jruby installed in pfnlt gemset `rvm install jruby-9.1.13.0@pdfnlt`
  - Parameters: Path to directory with PDFs to be analysed e.g. `../PDFNLT/pdfanalyzer/pdf/`
  - Description: Takes PDFs `from data/viewer_pdfs/`, copies them to `../PDFNLT/pdfanalyzer/pdf/` analyses them with PDFNLT to generate files like full text, split up sentences, sentence metadata, xhtml, etc.
  - Example script execution: `bash pdfnlt_find_occurrences.sh tse_ner ../PDFNLT/pdfanalyzer/pdf/`

- copy_pdfnlt_xhtmls_to_local_xhtmls.py:
  - Parameters: name of database data folder e.g. `tse_ner`
  - Description: Copies xhtml files as listed in `data/viewer_pdfs` from `../PDFNLT/pdfanalyzer/xhtml/` to `data/xhtml_raw/` and `data/xhtml_enriched` to prepare for entity extraction and xhtml enrichment for xhtmls
  - Example script execution: `python copy_pdfnlt_xhtmls_to_local_xhtmls.py tse_ner`

- scripts/find_entity_occurrences.py:
  - Parameters: name of database data folder e.g. `tse_ner`
  - Description: Enriches xhtmls in `data/xhtml_enriched` for papers in 'data/viewer_pdfs/' with metadata about entity occurences and generates entity occurence sets (entity list .txt file and occurrence metadata list .json file) and an overview file of entity occurrences for each facet in each analysed paper.
  - Example script execution: `python find_entity_occurrences.py tse_ner`

- generate_pdf_term_highlights_file.py:
  - Parameters: name of database data folder e.g. `tse_ner`
  - Description: Iterates over publication names in `data/xhtml_enriched/` folder and creates array of all term highlight objects (containing content, position and metadata information) for every term occurrence for every facet in selected publications. Outputs all concatenated highlights in `highlight/term-highlights.js`. Also outputs the list of viewer papers in `highlight/papers-list.js`.
  - Example script execution: `python generate_pdf_term_highlights_file.py tse_ner`

- Copy `highlight/term-highlights.js` and `highlight/papers-list.js` files to CIDV's `src/highlights/` folder.

## Contributing
Please feel free to contribute to the project by forking or creating a custom branch with a pull request. You can contact me on with any question, suggestions or other inquiries.

## License
Coner Document Analyser is [MIT LICENSED](https://github.com/vliegenthart/coner_document_analyser/blob/master/LICENSE).



