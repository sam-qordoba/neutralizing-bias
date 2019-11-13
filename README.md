# Neutralizing Biased Text

We provide code and documentation to accompany our work "Automatically Neutralizing Biased Text", to be presented at AAAI-20. This library provides discriminative models to detect biased language in Wikipedia articles, and generative models to generate 'de-biased' versions of biased sentences.

## Requirements 

```
pip install torch torchvision
pip install pytorch-pretrained-bert
pip install tensorboardX
pip install simplediff
pip install nltk
pip install sklearn
```

## Installation

```bash
pip install -r requirements.txt
```

## Overview 
Our code-based is structured in the following format: 

* `harvest/`: Provides utilities for crawling Wikipedia articles and for generating a parallel dataset of biased-debiased sentences. Our data generation approach mirrors that proposed by Recasens et al. (https://nlp.stanford.edu/pubs/neutrality.pdf). A final version of our crawled dataset can be found at https://stanford.io/2Q8G3bX. The zip file containing the data is 100MB
and expands to 500MB. 
* `src/`: This folder provides the model architectures, and training procedures for both detecting bias and generating 'debiased' versions of text. It is sub-divided in the following manner: 
    + `src/tagging/`: Functionality for detecting bias in a given input sentence. The model architectures, which are based on BERT and use the huggingface implementation, can be found under model.py. Simple baselines we implement, such as logistic regression classifiers, are presented in baseline.py. The primary training loop can be found under train.py. Utilities used by both model.py and train.py can be found under util.py.  To spawn a basic training and evaluation run, you can call the following from the root directory: 

    python tagging/train.py --train \<training dataset\> --test \<test dataset\> --working_dir \<dir\> --train_batch_size \<batch_size\> --test_batch_size \<batch_size\>  --hidden_size \<hidden_size\> --debug_skip

    By default, the tagging module is trained to incorporate the linguistic features enumerated by Recasens et al. For more information on how we incorporate these features into our BERT architecture, we direct you to the accompanying conference publication. In general terms, Marta's features comprise 32 linguistic features that are extracted for each word in a given sentence. In the process of training our model, we combine the BERT based word representation for each word with the words' accompanying linguistic features. We specify different ways in which these two representations can be combined, namely via concatenation or addition. We also allow users to specify whether the lingusitic features should be conatenated at the top or bottom of a given word's BERT embedding. These settings and the default specifications can be found under 'src/shared/args.py'. In addition to Recasens features, we also enable user to learn a category embedding, in addition to individual word embeddings, that specify a latent representation of the category of the article from which the input was extracted. These categories are derived from a set of predetermined article categories specified by the Wikipedia foundation. In our paper, we show empirically that conditioning our bias detection system on the given type of input enables the system to more accurately find bias. Setting and default specifications for jointly learning category embeddings can also be accessed under 'src/shared/args.py'.

    + `src/seq2seq/`: Functionality for generating debiased versions of a given biased sentence. As in the tagging directory, the model.py directory establishes the model architectures we use 
    in order to generate a debiased version for a given biased sentence. The models we establish are variants of basic Seq-2-Seq networks, with varying attention implementations. We provide an additional architecture of a generative debiasing model under transformer_decoder.py that uses a transformer-based architecture as a decoding module. The training procedure can again be found under train.py. To spawn a basic training and evaluation run, you can call the following from the root directory: 

    python seq2seq/train.py --train \<training dataset\> --test \<test dataset\> --working_dir \<dir\> --max_seq_len \<seq_size\> --train_batch_size \<batch_size\>  --test_batch_size \<batch_size\>   --hidden_size  \<hidden_size\> --debug_skip

    + `src/joint/`: Combines together the bias detection and debiasing modules into one end-to-end model that can be trained, and evaluated jointly. As in the other modules, the joint model architecture is stored under model.py and the primary training loop can be found under train.py. To spawn a basic training and evaluation run of our joint end-to-end framework, you can call the following from the root directory: 

    python joint/train.py --train  \<training dataset\> --test \<test dataset\>  --extra_features_top --pre_enrich --activation_hidden --tagging_pretrain_epochs 1 --pretrain_epochs 4     --learning_rate 0.0003 --epochs 2 --hidden_size 10 --train_batch_size 4 --test_batch_size 4     --bert_full_embeddings --debias_weight 1.3 --freeze_tagger --token_softmax --sequence_softmax  --working_dir <\dir\>  --debug_skip

    + `src/lexicons/`: Lexicons of words and their associated linguistic properties, such as impliocations, hedges, and factives. We require these lexicons to derive the features used by Recasens. et al to detect bias. 

    + `src/shared/`: A set of utilities that are shared by both the bias detection and debias generation modules, such as an implementation of beam search. We also store, constants and arguments that are shared globally. Args.py stores the entire set of arguments that can be passed into any one of the modules, along with a default specification. 
