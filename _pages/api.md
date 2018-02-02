---
title: Using the Stanford CoreNLP API 
keywords: api 
permalink: '/api.html'
---

## Generating annotations

The backbone of the CoreNLP package is formed by two classes: Annotation and Annotator. Annotations are the data structure which hold the results of annotators. Annotations are basically maps, from keys to bits of the annotation, such as the parse, the part-of-speech tags, or named entity tags. Annotators are a lot like functions, except that they operate over Annotations instead of Objects. They do things like tokenize, parse, or NER tag sentences. Annotators and Annotations are integrated by AnnotationPipelines, which create sequences of generic Annotators. Stanford CoreNLP inherits from the AnnotationPipeline class, and is customized with NLP Annotators.

The Annotators currently supported and the Annotations they generate are summarized [here](annotators.html).

To construct a Stanford CoreNLP object from a given set of properties, use `StanfordCoreNLP(Properties props)`. This method creates the pipeline using the annotators given in the "annotators" property (see below for an example setting). The complete list of accepted annotator names is listed in the first column of the table [here](annotators.html). To parse an arbitrary text, use the `annotate(Annotation document)` method.

```java
import edu.stanford.nlp.pipeline.*;
import java.util.*;

public class BasicPipelineExample {

    public static void main(String[] args) {

        // creates a StanfordCoreNLP object, with POS tagging, lemmatization, NER, parsing, and coreference resolution
        Properties props = new Properties();
        props.setProperty("annotators", "tokenize, ssplit, pos, lemma, ner, parse, dcoref");
        StanfordCoreNLP pipeline = new StanfordCoreNLP(props);

        // read some text in the text variable
        String text = "...";

        // create an empty Annotation just with the given text
        Annotation document = new Annotation(text);

        // run all Annotators on this text
        pipeline.annotate(document);

    }

}
```

You can give other properties to CoreNLP by building a Properties object
with more stuff in it. There are some overall properties like
`"annotators"` but most properties apply to one annotator and are
written as `annotator.property`. Note that **the value of a property is
always a String**. In our documentation of individual annotators, we
variously refer to their Type as "boolean", "file, classpath, or URL"
or "List(String)". This means that the String value will be
interpreted as objects of this type by using String parsing
methods. The value itself in the Properties object is always a String.
If you want to set several properties, you might find it
conventient to use our `PropertiesUtils.asProperties(String ...)`
method which will take a list of Strings that are alternately keys and
values and build a Properties object:

``` java
// build pipeline
StanfordCoreNLP pipeline = new StanfordCoreNLP(
	PropertiesUtils.asProperties(
		"annotators", "tokenize,ssplit,pos,lemma,parse,natlog",
		"ssplit.isOneSentence", "true",
		"parse.model", "edu/stanford/nlp/models/srparser/englishSR.ser.gz",
		"tokenize.language", "en"));

// read some text in the text variable
String text = ... // Add your text here!
Annotation document = new Annotation(text);

// run all Annotators on this text
pipeline.annotate(document);
```

If you do not anticipate requiring extensive customization, consider using the [Simple CoreNLP](simple.html) API.

## Interpreting the output

The output of the Annotators is accessed using the data structures CoreMap and CoreLabel. 

``` java
// these are all the sentences in this document
// a CoreMap is essentially a Map that uses class objects as keys and has values with custom types
List<CoreMap> sentences = document.get(SentencesAnnotation.class);

for(CoreMap sentence: sentences) {
  // traversing the words in the current sentence
  // a CoreLabel is a CoreMap with additional token-specific methods
  for (CoreLabel token: sentence.get(TokensAnnotation.class)) {
    // this is the text of the token
    String word = token.get(TextAnnotation.class);
    // this is the POS tag of the token
    String pos = token.get(PartOfSpeechAnnotation.class);
    // this is the NER label of the token
    String ne = token.get(NamedEntityTagAnnotation.class);
  }

  // this is the parse tree of the current sentence
  Tree tree = sentence.get(TreeAnnotation.class);

  // this is the Stanford dependency graph of the current sentence
  SemanticGraph dependencies = sentence.get(CollapsedCCProcessedDependenciesAnnotation.class);
}

// This is the coreference link graph
// Each chain stores a set of mentions that link to each other,
// along with a method for getting the most representative mention
// Both sentence and token offsets start at 1!
Map<Integer, CorefChain> graph = 
  document.get(CorefChainAnnotation.class);
```
