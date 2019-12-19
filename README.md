# Natural-Language-Processing
## Generating Natural Language Question-Answer Pairs

### Introduction

We have knowledge graphs like Freebase which collects the facts about the entities and relationships, which are used for answering fact-based questions. Whenever we ask a question to an assistant (Google Assistant or Siri), they go through these knowledge graphs and answer our questions. In our Project, we take these knowledge graphs and generate question-answer pairs. </br>
This kind of Question-Answer Generating system is used for several downstream applications. For example, instead of creating/updating QA pairs manually which takes a lot of labor and time, with some domain-specific knowledge graphs, We can generate QA pairs that can be used for developing quiz systems for educational purposes. We can even use these for the initial round of interviews.
We don't have any specific domain. We will be operating on an open dataset available from the WikiAnswers dataset. The goal of the project is to create the Question-Answer pairs. 

### Background

Generating both questions and answers pairs is very recent but in the past, there were many proposals for generating only questions from the text. Initially, the generation of questions started with a method called syntax-based method. This method parse the structure of sentences, identify key phrases and apply some known transformation rules to create questions. The transformation rules are nothing but semantic role labeling. These have limitations about irrelevant questions that are not related to the text and thus cannot be answered.
There are also template-based methods proposed where source text or a user query is taken as input, and a question template is prepared with placeholder variables which are to be replaced by the appropriate words from the input text to form a question. These keywords can be picked from the input text by identifying key phrases based on syntax-based method parsing.</br>
 Another approach used XML markup language to manually create question template and this might involve huge manual work in creating the templates. Later, a motivated approach is proposed in the paper “Automatically Generating Questions from Queries for Community-based Question Answering” to use millions of query questions to learn the question template pattern and adopt the pattern for the given input. Though this approach is having better question generalization performance, it can be limited to the application most of the time because of the training questions corpus.</br>
	In order to create an open domain model, we are going with a recent work proposed in the paper “Generating Natural Language Question-Answer Pairs from a Knowledge Graph Using an RNN Based Question Generation Model “. In this, we are going to use the Recurrent Neural Network (RNN) based model to generate questions from a set of keywords and the model can be trained using a dataset containing open domain keywords and question pairs.	
 
### Project
 
**Approach - **

The whole project of Question Answer pairs generation has fallen into three vital steps, one is Preprocessing the Freebase data to generate the test instances of the Text Generation model, the second is to generate the keywords for test instance and finally the last step is to implement a text generation model which is trained by a WikiAnswers dataset to generate questions from the keywords obtained from previous step.

**Preprocessing the data -**

	The knowledge graph we took is Freebase. We have done the following steps for preprocessing the knowledge graph. 
1.	Initially, the knowledge graph will be in RDF format. So, we had to preprocess the file containing RDF triples with URI links by removing the URI links and extracted the values like subject, predicate, object and predicate’s domain and range.</br>
2.	From these values, we made sure that process only unique predicates. We extracted unique subject, predicate and object, predicate pairs

Before processing, the data is in RDF format
<http://rdf.freebase.com/ns/american_football.football_player.footballdb_id>    <http://rdf.freebase.com/ns/type.object.type>   <http://rdf.freebase.com/ns/type.property> </br>
<http://rdf.freebase.com/ns/american_football.football_player.footballdb_id>	<domain>	<http://rdf.freebase.com/ns/american_football.football_player> </br>
<http://rdf.freebase.com/ns/american_football.football_player.footballdb_id>	<range>	<type.enumeration></br>
<http://rdf.freebase.com/ns/m.01001tl3>  <http://rdf.freebase.com/ns/music.recording.artist>        <http://rdf.freebase.com/ns/m.01s7hcz></br>
And here is how Processed Data is -</br>
Predicate List -</br>
<american_football.football_player.footballdb_id> </br> 
Domains and Ranges -   </br>
<american_football.football_player.footballdb_id>	<domain>	<american_football.football_player> </br>
<american_football.football_player.footballdb_id>	<range>	<type.enumeration></br>
Subject Predicate Object -</br>
<m.01001tl3>  <music.recording.artist>        <m.01s7hcz>	</br>

**Approach to generate QA pair for an Entity E -**

The Knowledge Graph(KG) contains information about various entities in the form of triples. A triple consists of a subject, predicate, and object. The subjects/objects(person, place, etc.) are the nodes of the knowledge graph whereas predicates are the edges of the KG. The predicates define the relationship between the subject and the object. To generate the QA pair, we need two modules.</br>
1.	Questions keywords and Answer Extractor: This is language independent and extracts required knowledge about the entity E from the KG.</br>
2.	RNN based Natural Language Question Generator: This is language-dependent and when fed with the information extracted from the first part it generates natural language QA pairs.

**Questions keywords and Answer Extractor - **

The keywords are a concise representation of the natural language question. For example, let's take the entity as “William Shakespeare”. We have natural language question as “Who was the author of the play Hamlet?”. The keywords identified are {‘Author’,’Play’,’Hamlet’}. We can have a Question and answer pair as ({‘Author’,’Play’,’Hamlet’},“William Shakespeare”). In the Knowledge Graph, “Author”, “Play”, “Hamlet” are the nodes that are connected to the entity “William Shakespeare”. We must first identify the entity node in the KG and get all its neighbors.</br>
Given a predicate pi, let sub(pi) be the subject of pi and obj(pi) be the object of pi. Let domain(pi) and range(pi) be the domain and range of pi respectively. Let {sub(pi), domain(pi), pi, obj(pi), range(pi)} be the 5-tuple associated with every pi. We use the following rules to generate QKA pairs from 5-tuples:</br>
1.	Unique Forward Relation</br>
2.	Unique Reverse Relation</br>

Generated Question Answer Keywords are as follows - 

Cantata misericordium, op. 69   form    compositional_form      Cantata</br>
Cantata form    composition     Cantata misericordium, op. 69</br>
Тацу    recording       release_track   Тацу</br>
Тацу    release release С тобой и без тебя</br>
С тобой и без тебя      release release_track   Тацу</br>
Over You        canonical_version       recording       Over You</br>
Over You        artist  artist  Jay Nash</br>
Jay Nash        artist  recording       Over You</br>
Dirty   release release Generation</br>
Generation      release release_track   Dirty</br>
Бенџамин Бритн  composer        composition     Cantata misericordium, op. 69</br>
Auf die Zechn-Tanz      composer        composer        Peter Havlicek</br>
Peter Havlicek  composer        composition     Auf die Zechn-Tanz</br>
Auf die Zechn-Tanz      recordings      recording       Auf die Zechn-Tanz</br>

**RNN based Natural Language Question Generator - **

As part of Natural Language question generator, we considered the question keywords that are identified in the previous step as input sequence and generate a question as output sequence. Instead of considering the set of keywords as a bag of words, we are giving importance to the order of occurrence of each keyword thereby making sure to generate semantically valid questions from the set of keywords. 
The model to generate question from set of keywords is inspired from RNN based encoder and decoder. In this approach, We used LSTM model, the keywords of different lengths are encoded to a fixed length vector representation and then compute the probability of the question output sequence for that given encoded vector representation. We feed the model with these output sequences and finally select the question with highest probability from all the generated questions. 

Sample Output: 

Input sentence: ticking sound alternator starter battery</br>
Decoded sentence: what to human part of the the battery is the battery

Input sentence: measurement is speed wind measured</br>
Decoded sentence: what measurement is wind speed measured east and in parallel what

Input sentence: jim morrison died</br>
Decoded sentence: jim jordans baseball what the s _END

Input sentence: mexico bigger australia</br>
Decoded sentence: how much can chemistry be considered in water _END

Input sentence: teachings muslim influenced arts</br>
Decoded sentence: cool quad procedures of flower in a and life in spain

Input sentence: bacterial infections cause cancer</br>
Decoded sentence: how chemistry is a creme brulee in _END

Input sentence: man loves woman</br>
Decoded sentence: about a soldier wear for a _END

### Results - 

We used BLEU score between the generated question and the reference question to evaluate the performance of Natural Language Generation system that we built. Here, there is only one reference question for each test Instance. The BLEU score here is an average n-gram overlap between the generated and reference question.</br>
Example generated text is -</br>
Input sentence: ticking sound alternator starter battery</br>
Decoded sentence: what to human part of the the battery is the battery</br>
Bleu score :  0.3686688 </br>
Overall Average BLEU Score obtained from all the test instances is 0.36875
 
References
-	Keras Documentation https://blog.keras.io/a-ten-minute-introduction-to-sequence-to-sequence-learning-in-keras.html</br>
-	Generating Natural Language Question-Answer Pairs from a Knowledge Graph using RNN Based Question Generation Model (https://www.aclweb.org/anthology/E17-1036)</br>
-	Freebase (https://en.wikipedia.org/wiki/Freebase)</br>
-	Stanford Tagger (https://nlp.stanford.edu/software/tagger.shtml)</br>
-	Wiki Answers Dataset (http://knowitall.cs.washington.edu/oqa/data/wikianswers/)</br>
-	Generating Natural Language Questions to Support Learning On-Line(https://www.aclweb.org/anthology/W13-2114)</br>
-	Automatically Generating Questions from Queries for Community-based Question Answering (https://www.aclweb.org/anthology/I11-1104)</br>
-	Learning Phrase Representations using RNN Encoder-Decoder for Statistical Machine Translation (https://arxiv.org/pdf/1406.1078.pdf)</br>
-	BLEU: a Method for Automatic Evaluation of Machine Translation (https://www.aclweb.org/anthology/P02-1040)</br>
-	BLEU Score https://www.nltk.org/_modules/nltk/translate/bleu_score.html</br>
