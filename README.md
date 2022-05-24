```python
teachers = ["manuel", "pablo", "leo", "violeta"]
teachers_key_words = {
    "manuel": ["starcraft", "build order", "optimization", "metaheuristics", "variable neighborhood search", "np-hard", "real time strategy", "iterated local lookup", "videogames", "rcpsp", "simulated annealing"],
    "pablo": ["compressed sensing", "image synthesis", "gpu", "alma", "radio interferometry", "cuda", "c++", "astronomy", "oop", "interferometry", "framework", "hpc", "gpgpu", "astroinformatics", "classification", "deep learning", "convolutional neural network", "big data"],
    "leo": ["bionic", "bmi", "deep learning", "artificial intelligence", "interfaces", "machine learning", "neuroscience", "neural networks", "spinal cord stimulation", "computational neuroscience", "parameter optimization", "evolutionary strategies", "cma-es", "electroretinogram", "alzheimer", "sample entropy", "fuzzy entropy", "complexity"],
    "violeta": ["deep learning", "convolutional neural networks", "classification", "human sperm heads", "morphology", "segmentation", "sperm", "deep cell", "gold standard", "transfer learning", "retina net", "panoptic"]
}
teachers_areas = {
    "manuel": ["applied informatics in biology and medicine", "computational biology bioinformatics", "informatics applied to industry", "advanced manufacturing", "informatics applied to education", "educational informatics"],
    "pablo": ["applied informatics in science"],
    "leo": ["applied informatics in biology and medicine", "computational neuroscience"],
    "violeta": ["applied informatics in biology and medicine", "medical imaging"],
}
# print(teachers_areas["manuel"])
```


```python
# extracting pdf data
import PyPDF2

pdf_file = open('ieee-taxonomy.pdf', 'rb')
read_pdf=PyPDF2.PdfFileReader(pdf_file)

pdf_text = ''
for i in range(1,read_pdf.getNumPages()):
    page = read_pdf.getPage(i)
    pdf_text = pdf_text + ' ' + page.extract_text()
    
pdf_text = pdf_text.replace("2022 IEEE Taxonomy", "")
pdf_text = pdf_text.replace("2022IEEE Taxonomy", "")
pdf_text = pdf_text.replace("This work is licensed under the Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 \nInternational License (CC BY-NC-ND 4.0).Created \nby The Institute ofElectrical and Electronics \n                             Engineers (IEEE) for the benefit of humanity.", "")
pdf_text = pdf_text.replace("IEEE Taxonomy: A Subset Hierarchical Display of IEEE Thesaurus Terms\nThe I EEE Taxonomy comprises t he f irst t hree hierarchical ' levels'  under each term-f amily (or branch)\nthat is formed from the top-most terms of the IEEE Thesaurus. In this document these term-familiesare \narranged alphabeticallyand denoted by boldface type. Each term family's hierarchy goes to nomore \nthan three sublevels, denoted by indents (in groupsof four dots) preceding the next level terms.A term \ncan appear in more than one hierarchical branch and can appear more than once in anyparticular \nhierarchy. The IEEE Taxonomy is defined in this wa\ny so that it is always a subset of the 2022IEEE\nThesaurus.", "")
pdf_text = pdf_text.replace("Page ", "")
pdf_text = pdf_text.replace(" ", "")
# print(pdf_text)
```


```python
# pdf text to pdf array
pdf_array_temp = pdf_text.split("\n")
pdf_array = []
for element in pdf_array_temp:
    if element != "" and not(element.isnumeric()):
        pdf_array.append(element)
# construir las categorias
pdf_categories = {}
temp_category = ""
for element in pdf_array:
    if element[0] != ".":
        temp_category = element
        pdf_categories[temp_category] = []
    else:
        pdf_categories[temp_category].append(element.replace(".", ""))

# for category in pdf_categories:
#     print(category)
```


```python
# utilizando similitudes entre palabras para poder lograr el match
from difflib import SequenceMatcher
def similar(a, b):
    return SequenceMatcher(None, a, b).ratio()
# print(similar("apple", "appel"))
```


```python
# recorrer las palabras claves de cada profesor para identificar su area
# sin "mineria de texto" al ser experimento
teachers_ieee_areas = {
    "manuel": [],
    "pablo": [],
    "leo": [],
    "violeta": []
} # teachers_areas son las sacadas desde la página de la usach

# n^4 no me enorgullece
for teacher in teachers_key_words:
    for word in teachers_key_words[teacher]:
        # por cada palabra clave necesito chequear en que categoria se encuentra
        # si es que tiene categoria, de ser así la agrego a las categorias en teachers_ieee_areas
        for category in pdf_categories:
            for to_compare in pdf_categories[category]:
                if (similar(word, to_compare) > 0.8): # 80% accurate to compare
                    try:
                        teachers_ieee_areas[teacher].index(category)
                        # se repite una categoria con este doctor
                    except ValueError:
                        teachers_ieee_areas[teacher].append(category)
                        teachers_ieee_areas[teacher].append(to_compare)
                    
teachers_ieee_areas # areas definidas por el tesauro con un 80% de certeza en la comparación de conceptos
```




    {'manuel': ['Mathematics', 'Optimization'],
     'pablo': ['Mathematics',
      'Compressedsensing',
      'accesspoints',
      'Imagesynthesis',
      'Systems,man,andcybernetics',
      'Humanimagesynthesis',
      'ScienceŒgeneral',
      'Astronomy',
      '(telecommunication)',
      'Neuroinformatics',
      'Professionalcommunication',
      'Neuroinformatics',
      'Computationalandartificialintelligence',
      'Deeplearning',
      'Systemsengineeringandtheory',
      'Deeplearning'],
     'leo': ['Computationalandartificialintelligence',
      'Deeplearning',
      'Systemsengineeringandtheory',
      'Deeplearning',
      'Systems,man,andcybernetics',
      'Userinterfaces',
      '(telecommunication)',
      'Geoscience',
      'Geoscienceandremotesensing',
      'Geoscience',
      'analysiscomputing',
      'Computationalneuroscience'],
     'violeta': ['Computationalandartificialintelligence',
      'Deeplearning',
      'Systemsengineeringandtheory',
      'Deeplearning',
      'accesspoints',
      'Imageclassification',
      'Systems,man,andcybernetics',
      'Morphology',
      '(telecommunication)',
      'Vegetation',
      'Lasersandelectrooptics',
      'Pigmentation',
      'Materials,elements,andcompounds',
      'Pigmentation']}




```python
# calcular la diferencia entre las areas existentes de la usach con las del tesauro

teachers_similarity = {
    "manuel": {
        "similarity": 0,
        "scores": []
    },
    "pablo": {
        "similarity": 0,
        "scores": []
    },
    "leo": {
        "similarity": 0,
        "scores": []
    },
    "violeta": {
        "similarity": 0,
        "scores": []
    }
}

# comparar areas todas con todas para ver su similitud
for teacher in teachers_ieee_areas:
    for ieee_area in teachers_ieee_areas[teacher]:
        for area in teachers_areas[teacher]:
            teachers_similarity[teacher]["scores"].append(similar(ieee_area, area))
            
# calcular el promedio de estas similitudes       
for teacher in teachers_similarity:
    teachers_similarity[teacher]["similarity"] = sum(teachers_similarity[teacher]["scores"]) / len(teachers_similarity[teacher]["scores"])

# teachers_similarity
print("Las areas de IEEE tienen una similud a las de la universidad de:\n")
for teacher in teachers_similarity:
    print(teacher+": "+str(round((teachers_similarity[teacher]["similarity"]*100), 2))+ "%")
```

    Las areas de IEEE tienen una similud a las de la universidad de:
    
    manuel: 31.41%
    pablo: 28.25%
    leo: 31.55%
    violeta: 25.91%



```python

```
