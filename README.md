## SCODE Word Embeddings using Substitute Words##

This repository provides a tool to induce word vectors using substitute words.
Using this repository you can:

- Generate word-type embeddings described in [1,3,4]
- Generate word-token embeddings described in [2,5]

[1]. Learning Syntactic Categories Using Paradigmatic Representations of Word Context, In Proceedings of the 2012 Conference on Empirical Methods in Natural Language Processing (EMNLP-CONLL 2012), Jeju, Korea, July. Association for Computational Linguistics, [Paper, Presentation & Code](http://www.denizyuret.com/2012/05/learning-syntactic-categories-using.html), [bib](http://aclweb.org/anthology/D/D12/D12-1086.bib). 

[2]. Word Context and Token Representations from Paradigmatic Relations and Their Application to Part-of-Speech Induction, [Paper & Presentation](http://www.denizyuret.com/2013/09/enis-rfat-sert-ms-2013.html)

[3]. The AI-KU System at the SPMRL 2013 Shared Task : Unsupervised Features for Dependency Parsing, In Proceedings of the Fourth Workshop on Statistical Parsing of Morphologically-Rich Languages, pp 78--85, Seattle, Washington, USA, October. Association for Computational Linguistics, [Paper](http://aclweb.org/anthology//W/W13/W13-4909.pdf), [Word Embeddings](https://github.com/wolet/sprml13-word-embeddings), [bib](http://aclweb.org/anthology//W/W13/W13-4909.bib)

[4] Substitute Based SCODE Word Embeddings in Supervised NLP Tasks, [Word Vectors For 7 Languages](https://drive.google.com/folderview?id=0ByseGoDIuMC6eXpaQWZNZWdxT1E&usp=sharing), [bib](https://www.cs.cmu.edu//~vcirik/assets/bibs/arxiv2014.bib)

### Other Word Vectors

Here is a list of other word vectors : 
- Turian, [Word representations: A simple and general method for semi-supervised learning](http://metaoptimize.com/projects/wordreprs/)
- Dhillon, [Multi-View Learning of Word Embeddings via CCA](http://www.pdhillon.com/publications.html)
- Mikolov, [Recurrent neural network based language model](http://www.fit.vutbr.cz/~imikolov/rnnlm/)
- Mikolov, [Efficient Estimation of Word Representations in Vector Space](https://code.google.com/p/word2vec/)
- Huang, [Improving Word Representations Via Global Context And Multiple Word Prototypes](http://www.socher.org/index.php/Main/ImprovingWordRepresentationsViaGlobalContextAndMultipleWordPrototypes)
- Stratos, [A Spectral Algorithm for Learning Class-Based n-gram Models of Natural Language](https://github.com/karlstratos/cca)
- Yogatama, [Learning Word Representations with Hierarchical Sparse Coding](http://www.ark.cs.cmu.edu/dyogatam/wordvecs/)
- Faruqui, [Improving Vector Space Word Representations Using Multilingual Correlation](https://github.com/mfaruqui/eacl14-cca)
- Lebret, [Word Embeddings through Hellinger PCA](http://lebret.ch/words/)

#### Semi-Supervised Word Vectors

Below is a list of word vectors using dependency pairs for inducing representations.

- Murphy, [Learning Effective and Interpretable Semantic Models using Non-Negative Sparse Embedding](http://www.cs.cmu.edu/~bmurphy/NNSE/)
- Levy, [Dependency-Based Word Embeddings](http://levyomer.wordpress.com/2014/04/25/dependency-based-word-embeddings/)
- Bansal, [Tailoring Continuous Word Representations for Dependency Parsing](ttic.edu/bansal/data/syntacticEmbeddings.zip )

 
## 0 - Setting-Up The Environment

After cloning the repository, the first thing to do is initializing the binary directory.
Simlpy go to /run directory and run this command: 

     make bin

This will clones some repositories needed for a successfull run and install them. It will
also copy the script files unde /bin directory.

Then, in order to generate word substitutes you need a Language Model(LM) in SRILM format.
Install [SRILM](http://www.speech.sri.com/projects/srilm/download.html) and set the SRILM_PATH variable in the Makefile.

A one-liner procuding word vectors is as follows, just change the ones in the capital letters accordingly: 

    zcat ../data/YOUR_CORPUS.tok.gz | fastsubs-omp -n 100  -p 1.0 YOUR_LM.lm.gz | grep -v '^</s>' | wordsub-n -s 1 -n 100  | scode-online -v2 -d NUMBER_OF_DIMENSION -s 1  | perl -ne 'print if s/^0://' | cut -f1,3- | tr '\t' ' ' | gzip > YOUR_WVEC.gz

This target is at the end of the run/Makefile . But, one may need a detailed description about what's going on. Let's move on.

## 1 - Generate Type & Token Vectors - A Shortcut:

This section here to generate word embeddings embeddings right away.
However, if you want to know how they are generated please skip
this section. We provide a test file named mini-{train|test}.tok.gz so that
you can test your setup.

Use an LM Corpus(e.g. Wikipedia) and a target corpus(e.g CONLL Task) to generate 25-dimension word embeddings:

    make -n YOUR-TARGET-CORPUS.unk-type-25.gz LMFILE=YOUR-LM-CORPUS DIM=25

For word-token vectors, you can use 4 different methods,

    make YOUR-TARGET-CORPUS.XsubX.gz LMFILE=YOUR-LM-CORPUS DIM=25 X1=YOUR-TARGET-CORPUS.unk-type-25.gz X2=YOUR-TARGET-CORPUS.unk-type-25.gz 

    make YOUR-TARGET-CORPUS.XplusX.gz LMFILE=YOUR-LM-CORPUS DIM=25 X1=YOUR-TARGET-CORPUS.unk-type-25.gz X2=YOUR-TARGET-CORPUS.unk-type-25.gz 

    make YOUR-TARGET-CORPUS.XplusY.gz LMFILE=YOUR-LM-CORPUS DIM=25 X1=YOUR-TARGET-CORPUS.unk-type-25.gz X2=YOUR-TARGET-CORPUS.unk-type-25.gz 

    make YOUR-TARGET-CORPUS.knn.XY.gz LMFILE=YOUR-LM-CORPUS DIM=25 X1=YOUR-TARGET-CORPUS.unk-type-25.gz X2=YOUR-TARGET-CORPUS.unk-type-25.gz 

or you can combine your favorite word embeddings:

    make YOUR-TARGET-CORPUS.XmixX.gz LMFILE=YOUR-LM-CORPUS DIM=25 X1=YOUR-EMBEDDINGS1.gz X2=YOUR-EMBEDDINGS2.gz 

## 2 - How to Generate Word-Type Embeddings?

First train an LM using a large corpus. Tokenize your corpus 
into YOUR-LM-CORPUS.tok and place it under /data and gzip tokenized 
file into YOUR-LM-CORPUS.tok.gz.

Then run:

    make YOUR-LM-CORPUS.lm.gz

You may want to change LM related variables in the Makefile such as LM_DISCOUNT, LM_NGRAM etc.

Then generate substitute words. First put your substitute(target) corpus
under /data in a similar way you did to LM corpus, YOUR-SUB-CORPUS.tok.gz. Your substitute corpus and
LM corpus may differ. But use **the same tokenization**. Then, you should run:

    make YOUR-SUB-CORPUS.sub.gz LM=YOUR-LM-CORPUS

Again you may want to change word substitute options like FS_NSUB, FS_PSUB. Note that, fastsubs-omp
can be run in parallel, thus , you can set number of threads with OMP_NUM_THREADS variable. We
observe no gain after 24 core.

Now you can generate <word,substitute word> pairs using substitute distributions. You
can change the number of substitutes and random seed by changing corresponding
variables(i.e WORDSUB,SEED). Just run:

     make YOUR-SUB-CORPUS.pairs.gz

If you want to generate an embedding for unknown words (probably you do) with an unknown tag \*UNKNOWN\*:

     make YOUR-SUB-CORPUS.unk-pairs.gz

Now you can generate word-type (one embedding per word) embeddings. Let's say
you want 25-dimension word embeddings with unknown word tag. Run:

    make YOUR-SUB-CORPUS.unk-type-25.gz DIM=25

Or you can generate for all words (without unknown tag):

    make YOUR-SUB-CORPUS.type-25.gz DIM=25

It runs SCODE, then extracts word-type embeddings. Note that, you can 
change a variety of parameters of SCODE.

If you only interested in word-type embeddings you are good to go.

## 3 - How to Generate Word-Token (Context-Dependent) Embeddings?

After generating word-type embeddings, you can generate context
dependent word embeddings in a couple of different ways.

### Enis Sert's KNN Based Vectors(2):

First find the k-nearest-neighbors(k=128).

    make YOUR-SUB-CORPUS.knn128.gz

As always there are variables that you can change here such as KNN_METRIC -- number of nearest
neighbors.

Run following command to generate 50-dimension word-token vectors:

    make YOUR-SUB-CORPUS.knn.XY.gz DIM=25

### Substitute Pairs Based Vectors:

#### Using Substitute Word Embeddings of SCODE Sphere

You can use <word,substitue word> pairs (<X,Y> where X is target word, Y is substitute word)
and substite word embeddings of SCODE sphere to generate word-token
embeddings.

After generating unk-pairs.gz and 25-dimension word-type embeddings, just run:

    YOUR-SUB-CORPUS.XplusY.gz DIM=25

Or, you can use <word,substitue word> pairs and word embeddings of
target words for substitute words:

    YOUR-SUB-CORPUS.XplusX.gz DIM=25

These will generate 25+25-dimension word-token vectors.

#### Using Any Word Embeddings You Like

You can combine two different word embeddings using <word,substitute word>
pairs to generate word-token embeddings. First generate unk-pair.gz
then gzip your favorite word embeddings [1](https://code.google.com/p/word2vec/),[2](http://metaoptimize.com/projects/wordreprs/) which should contain \*UNKNOWN\* tag for unknown words and run:

    make YOUR-SUB-CORPUS.XmixX.gz X1=YOUR-EMBEDDING1.gz X2=YOUR-EMBEDDING2.gz

Of course you can use the same word embedding for X1 and X2.

### Substitute Distribution Based Vectors:

The generalization of the previous method is using substitute
distribution instead of <word,substitute word> pairs.

    make YOUR-SUB-CORPUS.XsubX.gz X1=YOUR-EMBEDDING1.gz X2=YOUR-EMBEDDING2.gz

## 4 - Using Morphologic and Orthographic Features

If you want to generate word embeddings using Orthographic and Morphologic
features using [Morfessor](http://www.cis.hut.fi/projects/morpho/), we suggest
you use a big corpus(maybe your LM corpus) and run:

    make YOUR-FEAT-CORPUS.feat.gz

It will generate these features. Please take a look at Morfessor parameters in
the Makefile. Than simply append **"+f"** to your target file, set **FEATUREFLAG=+f**
and **FEATFILE=YOUR-FEAT-CORPUS**:
       
<code> make YOUR-SUB-CORPUS.unk-type<b>+f</b>.gz  DIM=25 **FEATUREFLAG=+f** **FEATFILE=YOUR-FEAT-CORPUS**</code>

or

<code> make YOUR-SUB-CORPUS.XplusY<b>+f</b>.gz DIM=25 **FEATUREFLAG=+f** **FEATFILE=YOUR-FEAT-CORPUS**</code>

or

<code> make YOUR-SUB-CORPUS.knn.XY<b>+f</b>.gz DIM=25 **FEATUREFLAG=+f** **FEATFILE=YOUR-FEAT-CORPUS**</code>

    
## TODO:

- Write a better README.
- Word Features
