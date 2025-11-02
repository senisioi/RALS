# RALS: Resources and Baselines for Romanian Automatic Lexical Simplification

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
[![Paper](https://img.shields.io/badge/paper-ACL--style-orange)](https://aclanthology.org/2025.emnlp-main.1603/)
[![Dataset Status](https://img.shields.io/badge/dataset-released-green)](https://github.com/senisioi/RALS/raw/refs/heads/main/data.zip)
[![Language](https://img.shields.io/badge/language-Romanian-red)]()



This repository accompanies the paper "RALS: Resources and Baselines for Romanian Automatic Lexical Simplification", which introduces the first set of resources and baseline systems for Lexical Complexity Prediction (LCP) and Lexical Simplification (LS) in Romanian.
The project provides new datasets, new evaluation protocols, and several baseline systems.

**Contributors**: [Fabian Anghel](https://github.com/Bagsylina/), [Petru-Theodor Cristea](https://github.com/petruTH/), Claudiu Creangă, and Sergiu Nisioi

## Romanian Lexical Complexity Prediction (LCP) Data

The lexical complexity annotations for Romanian are in a password protected archive (pw: ralsresources). Hopefully to avoid trivial LLM contamination.

Data suffixed with MLSP is translated from the 2024 Shared Task on [Multilingual Lexical Simplification Pipeline (MLSP)](https://sites.google.com/view/mlsp-sharedtask-2024/home) at [BEA2024](https://sig-edu.org/bea/2024).

The archive contains the following files:

- **ht_from_english_MLSP** - Human Translations (HT) from MLSP English; the data is clean, human annotations have a good agreement; 5 annotations per sample
- **wt_original** - Word Translations (WT) into Romanian from the English MLSP data; sentences containing the words are searched in [The Romanian Language Repository](https://github.com/lmidriganciochina/romaniancorpus); the embeddings of each word are clustered automatically and we select sentences [close and far apart from the centroid](https://github.com/Bagsylina/sentence_pairs_annotation/blob/dea06c8ff1382b6287d4f47c15196ec4af086ab7/contextual_word_clustering/words_clustering.py#L98), leading to different meanings and usages of the same word
- **RoLCP** - data annotated by humans with high agreements 10 annotations per sample; the texts have different genres and word selection process is done based on [wordfreq](https://pypi.org/project/wordfreq/)
- additionally we provide Machine Translations (mt_from_MLSP.csv) from all languages into Romanian; errors may be there, we removed all sentences containing invalid Romanian target words; does not have human LCP annotations 

The English and Romanian Human Translation (HT) sentences (569 samples), the Romanian Word-level translation (WT) datasets (1765 samples), and the new RoLCP data (1,587 samples) have in total, 3,921 annotated samples for LCP on Romanian. The annotated complexity for words occurring in original texts resemble closer the distribution of the similar word annotations in original English:

|                 | **Sentence length – En** | **HT** | **WT** | **RoLCP** | **Complexity – En** | **HT** | **WT** | **RoLCP** |
|-----------------|---------------------------|--------|--------|-----------|----------------------|--------|--------|-----------|
| **mean**        | 22.82                     | 24.46  | 24.53  | 27.58     | 0.24                 | 0.13   | 0.28   | 0.23      |
| **std**         | 7.74                      | 8.62   | 8.43   | 26.59     | 0.19                 | 0.25   | 0.21   | 0.23      |
| **min**         | 7                         | 9      | 6      | 2         | 0.02                 | 0      | 0      | 0         |
| **max**         | 45                        | 49     | 42     | 318       | 0.93                 | 1      | 1      | 1         |
| **no. samples** | 569                       | 569    | 1,765  | 1,587     | 569                  | 569    | 1,765  | 1,587     |
| **no. sentences** | 190                     | 190    | 751    | 274       | 190                  | 190    | 751    | 274       |



The word translation (WT) sentence selection process is in an unstructured form [in this repository](https://github.com/Bagsylina/sentence_pairs_annotation/tree/main). Please reach out if you need help understanding the code or extracting sentences for other languages.


## DexFlex 

[DexFlex](https://github.com/PetruTH/DexFlex) is a Python package for Romanian language processing.
It leverages [dexonline](https://github.com/dexonline/dexonline) to extract dictionary data, applies rule-based methods, and integrates **spaCy** with the Romanian model [`ro_core_news_lg`](https://spacy.io/models/ro).  
The library provides functionality such as inflection handling, synonym suggestions, and transformations between tenses and voices.

The system is rule based and has the following steps:

1. Identifies part-of-speech & morphology using spaCy.  
2. Retrieves synonyms from the dexonline dictionary.  
3. Performs meaning disambiguation using contextual embeddings.  
4. Inflects synonyms correctly to match Romanian grammar.  
5. Ranks candidates using LCP scores.

To test the package, try:

```bash
pip install dexflex
```

```python
from dexflex.prototype import *
# ! when running this for the first time
# it will download the dictionary data ~1GB
# in the location where the script is run

import spacy

nlp = spacy.load("ro_core_news_lg")

samples = [
    ("Am observat eroarea la timp.", 2),
    ("Ea a învățat să cânte la pian.", 4),
    ("Am observat eroarea la timp.", 1),
    ("Ei au reparat mașina stricată.", -2),
    ("Tu ai scris o poezie frumoasă.", -2),
    ("Ea s-a gătit pentru o cină delicioasă.", 3),
    ("El a uitat să aducă documentele.", -2),
    ("Ea a primit un cadou de ziua ei.", 4),
    ("Tu ai vorbit cu profesorul?", -2),
    ("Ei au construit un parc nou.", 2),
    ("El a plecat la camera lui.", -3)
]

for sample in samples:
    doc = nlp(sample[0])
    syn_list = [syn[0] for syn in doc[sample[1]]._.get_syns()]
    print(f"Synonyms for: {doc[sample[1]]} are {syn_list}")


```
```
Synonyms for: eroarea are ['eroarea', 'lipsa', 'greșeala', 'rătăcirea', 'necinstea', 'înșelătoria', 'incorecțiunea', 'neregula', 'inexactitatea', 'șarlatania']
Synonyms for: cânte are ['cânte', 'execute', 'doinească', 'sune', 'horească', 'răsune', 'intoneze', 'fredoneze']
Synonyms for: observat are ['văzut', 'observat', 'surprins', 'prins', 'simțit', 'constatat', 'aflat', 'privit', 'sesizat', 'priceput']
Synonyms for: stricată are ['stricată', 'vicioasă', 'spartă', 'uzată', 'primejduită', 'degradată', 'zdrobită', 'atinsă', 'deformată', 'alterată']
Synonyms for: frumoasă are ['frumoasă', 'arătoasă', 'frumoaso', 'chipoasă', 'aleasă', 'distinsă', 'drăguță', 'minunată', 'aspectuoasă', 'strașnică']
Synonyms for: gătit are ['gătit', 'dichisit', 'ferchezuit', 'sclivisit', 'înfrumusețat', 'înzorzonat', 'împopoțat', 'înțoțonat', 'împoponat', 'zorzonat']
Synonyms for: documentele are ['actele', 'documentele', 'izvoarele', 'textele', 'înscrisurile', 'zapisele', 'hrisoavele', 'titlurile', 'terfeloagele']
Synonyms for: cadou are ['dar', 'plocon', 'omagiu']
Synonyms for: profesorul are ['profesorul', 'dascălul', 'profesoru', 'dascălu', 'dăscălașul', 'dăscălașu', 'pedagogul', 'pedagogu', 'profesorașul', 'profesorașu']
Synonyms for: construit are ['făcut', 'clădit', 'construit', 'temeiat', 'compus', 'edificat', 'constituit', 'întemeiat', 'conceput', 'creat']
Synonyms for: camera are ['camera', 'odaia', 'cuhnia', 'încăperea']
```


## Lexical Simplification & DexFlex & GPT-4o

- Built from the HT subset.
- Annotators propose synonyms without ranking them.
- All synonym pairs are evaluated using pairwise simplicity judgments.
- Final rankings computed via a Bradley–Terry model.

We evaluate five systems:

- **Apertus-8B** (multilingual open LLM)
- **RoLlama-8B** (Romanian-instruction tuned)
- **GPT-4o**
- **Romanian BERT** classifier
- **DexFlex** (hybrid synonym + WSD + inflection)

| Metric | Apertus | RoLlama | DexFlex | GPT-4o | BERT-Ro |
|--------|---------|---------|----------|---------|----------|
| **MAP@1** | 0.21 | 0.40 | **0.41** | 0.28 | 0.27 |
| **MAP@5** | 0.10 | 0.16 | **0.30** | 0.15 | 0.15 |
| **Potential@5** | 0.33 | 0.51 | **0.67** | 0.51 | 0.48 |
| **ACC@1 (top gold)** | 0.11 | **0.22** | 0.15 | 0.16 | 0.14 |

DexFlex consistently outperforms all other approaches across MAP and Potential metrics, showing strong robustness for synonym generation. While RoLlama achieves slightly better accuracy on top-gold metrics, it lags behind in overall ranking and coverage. 

A hybrid approach like DexFlex can be more effective than large general-purpose LLMs or fine-tuned BERT, especially when training on small datasets.

[The scripts for generating simplification results using DexFlex and GPT-4o](https://github.com/PetruTH/emnlp2025_rals) and the evaluation metrics are available in a different repository.

Additional experiments using BERT-based models are available [here](https://github.com/Bagsylina/Romanian-Text-Simplification)


## Cite
```
@inproceedings{anghel-etal-2025-rals,
    title = "{RALS}: Resources and Baselines for {R}omanian Automatic Lexical Simplification",
    author = "Anghel, Fabian  and
      Petru-Theodor, Cristea  and
      Creanga, Claudiu  and
      Nisioi, Sergiu",
    editor = "Christodoulopoulos, Christos  and
      Chakraborty, Tanmoy  and
      Rose, Carolyn  and
      Peng, Violet",
    booktitle = "Proceedings of the 2025 Conference on Empirical Methods in Natural Language Processing",
    month = nov,
    year = "2025",
    address = "Suzhou, China",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/2025.emnlp-main.1603/",
    pages = "31469--31480",
    ISBN = "979-8-89176-332-6",
}
```


