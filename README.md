# Fuseki reasoning example

We would like to use Fuseki in the following way:
- A rule-based reasoner configured with custom rules uses all named graphs and adds the results to the union graph.
- We can use SPARQL updates to load/update named graphs; this should trigger the rule-based reasoner to update the results in the union graph.
- We can use SPARQL queries against the union graph or named graphs.

How do we achieve the above?

This example is an experiment to explore how to achieve the above using the Fuseki API.
Once we managed to do this, the next question is how to achieve this functionality via a Fuseki server configuration.

To run in IntelliJ, use the 'OwlReasonIncrementallyJena1d' run configuration.

It is equivalent to executing:

```shell
java \
-Dlog4j.debug=false -Dlog4j.configurationFile=classpath:log4j2.properties \
io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d \
-c owl-fuseki-reasoner/src/main/resources/owl-ventailments/catalog.xml
```

Note: There are variations of this code that attempt at using the Jena APIs for the union graph instead of constructing a union explicitly.
These alternatives are unsuccessful in that the queries before SPARQL updates do not produce the expected results.
The differences with [OwlReasonIncrementallyJena1d](java/io/opencaesar/owl/fuseki_reasoner/OwlReasonIncrementallyJena1d.java) 
are minimized to facilitate comparison.
- [OwlReasonIncrementallyJena1e](java/io/opencaesar/owl/fuseki_reasoner/OwlReasonIncrementallyJena1e.java)
- [OwlReasonIncrementallyJena1f](java/io/opencaesar/owl/fuseki_reasoner/OwlReasonIncrementallyJena1f.java)

This code does the following:

## 1) Create an in-memory TDB2 dataset.

```java
Dataset ds0 = TDB2Factory.createDataset();
```

## 2) Load various ontologies in OWL and TTL serialization.

```java
for (var iri : options.inputOntologyIris) {
    Txn.executeWrite(ds0, () -> {
        Model m = ModelFactory.createDefaultModel();
        fm.readModelInternal(m, iri);
        ds0.addNamedModel(iri, m);
        LOGGER.info("Loading named graph: " + iri);
    });
}
```

## 3) create a union model.

```java
Model unionModel = ModelFactory.createDefaultModel();
Txn.executeRead(ds0, () -> {
    Iterator<Resource> it = ds0.listModelNames();
    while (it.hasNext()) {
        Resource r = it.next();
        unionModel.add(ds0.getNamedModel(r));
    }
});
Txn.executeWrite(ds0, () -> {
    ds0.setDefaultModel(unionModel);
});
```

So far, this corresponds to the following output:

<details>

```text
Logger Factory: org.apache.logging.slf4j.Log4jLoggerFactory
15:21:12.847 [main] DEBUG org.apache.jena.info - System architecture: 64 bit
15:21:12.883 [main] DEBUG org.apache.jena.dboe.System - System architecture: 64 bit
15:21:12.937 [main] DEBUG io.micrometer.common.util.internal.logging.InternalLoggerFactory - Using SLF4J as the default logging framework
15:21:13.058 [main] DEBUG org.apache.jena.util.FileManager - Add location: LocatorFile
15:21:13.059 [main] DEBUG org.apache.jena.util.FileManager - Add location: LocatorURL
15:21:13.060 [main] DEBUG org.apache.jena.util.FileManager - Add location: ClassLoaderLocator
15:21:13.065 [main] DEBUG org.apache.jena.util.FileManager - Found: ont-policy.rdf (ClassLoaderLocator)
15:21:13.545 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - Adding local file mapping for: http://example.com/tutorial/vocabulary/bundle/classes
15:21:13.545 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - Adding local file mapping for: http://example.com/tutorial/vocabulary/mission
15:21:13.545 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - Adding local file mapping for: http://example.com/tutorial/description/una1
15:21:13.545 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - Adding local file mapping for: http://example.com/tutorial/vocabulary/bundle/properties
15:21:13.545 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - Adding local file mapping for: http://example.com/tutorial/description/bundle
15:21:13.545 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - Adding local file mapping for: http://example.com/tutorial/vocabulary/bundle/individuals
15:21:13.545 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - Adding local file mapping for: http://example.com/tutorial/vocabulary/bundle
15:21:13.552 [main] DEBUG org.apache.jena.info - File mode: Mapped
15:21:13.620 [main] DEBUG org.apache.jena.tdb2.store.TDB2StorageBuilder - Triple table: SPO :: SPO,POS,OSP
15:21:13.628 [main] DEBUG org.apache.jena.tdb2.store.TDB2StorageBuilder - Quad table: GSPO :: GSPO,GPOS,GOSP,POSG,OSPG,SPOG
15:21:13.634 [main] DEBUG org.apache.jena.tdb2.store.TDB2StorageBuilder - Prefixes: GPU :: GPU
15:21:13.728 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - Loading named graph: http://example.com/tutorial/vocabulary/bundle/classes
15:21:13.773 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - Loading named graph: http://example.com/tutorial/vocabulary/mission
15:21:13.786 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - Loading named graph: http://example.com/tutorial/description/una1
15:21:13.797 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - Loading named graph: http://example.com/tutorial/vocabulary/bundle/properties
15:21:13.802 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - Loading named graph: http://example.com/tutorial/description/bundle
15:21:13.810 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - Loading named graph: http://example.com/tutorial/vocabulary/bundle/individuals
15:21:13.816 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - Loading named graph: http://example.com/tutorial/vocabulary/bundle
```

</details>

## 4) What can we get from SPARQL queries?

```java
Txn.executeRead(ds0, () -> {
    LOGGER.info("ds0: Check how many results we get querying named graphs.");
    queryString("SELECT ?g ?s ?p ?o { GRAPH ?g { ?s ?p ?o} }", ds0, false);
    LOGGER.info("ds0: Check how many results we get querying the union graph.");
    queryString("SELECT * {?s ?p ?o}", ds0, false);
});
```

The results seem fine.

<details>

```text
15:21:13.842 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - ds0: Check how many results we get querying named graphs.
15:21:13.842 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: SELECT ?g ?s ?p ?o { GRAPH ?g { ?s ?p ?o} }
15:21:14.010 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (251 results)
15:21:14.010 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - ds0: Check how many results we get querying the union graph.
15:21:14.010 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: SELECT * {?s ?p ?o}
15:21:14.012 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (182 results)
```

</details>

## 5) We want to query the union graph w/ the reasoner entailments.

```java
Model baseModel = ds0.getDefaultModel();

Resource cr = ModelFactory.createDefaultModel().createResource();
cr.addProperty(ReasonerVocabulary.PROPderivationLogging, "true");
cr.addProperty(ReasonerVocabulary.PROPenableOWLTranslation, "true");
cr.addProperty(ReasonerVocabulary.PROPenableTGCCaching, "true");
cr.addProperty(ReasonerVocabulary.PROPtraceOn, "false");
cr.addProperty(ReasonerVocabulary.PROPruleMode, GenericRuleReasoner.HYBRID.toString());
cr.addProperty(ReasonerVocabulary.PROPruleSet, "owl-fuseki-reasoner/src/main/resources/mission.rules");
Reasoner gr = GenericRuleReasonerFactory.theInstance().create(cr);

InfModel infModel = ModelFactory.createInfModel(gr, baseModel);

// Wrapping the infModel results in an unsupportedMethod exception when executing SPARQL queries.
// Dataset ds1 = DatasetFactory.wrap(infModel);
Dataset ds1 = DatasetFactory.create(infModel);
```

There are many APIs for creating and wrapping datasets. Some combinations work, others produce errors.

## 6) Let's run some queries:

```java
Txn.executeRead(ds1, () -> {
    LOGGER.info("before insertion ds1: Check how many results we get querying named graphs.");
    queryString("SELECT ?g ?s ?p ?o { GRAPH ?g { ?s ?p ?o} }", ds1, false);
    LOGGER.info("before insertion ds1: Check how many results we get querying the union graph.");
    queryString("SELECT * {?s ?p ?o}", ds1, false);
```

Why does the named graph query produce no results instead of 251 as before?

Why does the union graph query produce only 214 results instead of 182 as before?

```text
15:21:14.094 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - before insertion ds1: Check how many results we get querying named graphs.
15:21:14.094 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: SELECT ?g ?s ?p ?o { GRAPH ?g { ?s ?p ?o} }
15:21:14.099 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (0 results)
15:21:14.099 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - before insertion ds1: Check how many results we get querying the union graph.
15:21:14.099 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: SELECT * {?s ?p ?o}
15:21:14.135 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (214 results)
```

## 7) Check reasoner entailments.

```java
LOGGER.info("before insertion ds1: Check named graphs for patterns: ?x mission:presents ?y.");
queryPresentsByGraph(ds1, true);
LOGGER.info("before insertion ds1: Check union graph for patterns: ?x mission:presents ?y.");
queryPresentsByUnion(ds1, true);
```

These queries depend on the rule we loaded in the reasoner: [owl-fuseki-reasoner/src/main/resources/mission.rules](owl-fuseki-reasoner/src/main/resources/mission.rules); in particular:

```
@prefix mission: <http://example.com/tutorial/vocabulary/mission#>
@prefix oml: <http://opencaesar.io/oml#>

[missionPresents:
    (?r rdf:type mission:Presents),
    (?r oml:hasSource ?s),
    (?r oml:hasTarget ?t)
    ->
    (?s mission:presents ?t)
]
```

Querying named graphs should produce no results because the entailments go in the union graph.

```text
15:21:14.135 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - before insertion ds1: Check named graphs for patterns: ?x mission:presents ?y.
15:21:14.135 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: PREFIX mission: <http://example.com/tutorial/vocabulary/mission#>
SELECT ?g ?c ?i WHERE { GRAPH ?g { ?c mission:presents ?i . } }
ORDER BY ?g ?c ?i
15:21:14.148 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (0 results)
```

Querying the union graph provides evidence that the rules produced the expected derived triples.

```text
15:21:14.149 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - before insertion ds1: Check union graph for patterns: ?x mission:presents ?y.
15:21:14.149 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: PREFIX mission: <http://example.com/tutorial/vocabulary/mission#>
SELECT ?c ?i WHERE { ?c mission:presents ?i . }
ORDER BY ?c ?i
15:21:14.162 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  c=http://example.com/tutorial/description/una1#C1 i=http://example.com/tutorial/description/una1#I1
15:21:14.162 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  c=http://example.com/tutorial/description/una1#C2 i=http://example.com/tutorial/description/una1#I2
15:21:14.162 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  c=http://example.com/tutorial/description/una1#C3 i=http://example.com/tutorial/description/una1#I3
15:21:14.162 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (3 results)
```

Given the reasoner options in (5), there should be log messages about the rule derivations; yet there are none.
Why?


Counting statements in the base graph produces the same result as counting the triples from sparql.
However, the statement count for the inference graph is yet a different total, 208, compared to the sparql query, 214.
What's the difference?

```text
15:21:14.163 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - valid = true
15:21:14.164 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - statements (base) = 182
15:21:14.165 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - statements (inf)  = 208
```

## 8) Testing a sparql insert update.

The named graph query produces 4 triples; this is consistent with the insert update.

```text
15:21:14.173 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - INSERT...
15:21:14.180 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - after insertion ds1: Check how many results we get querying named graphs.
15:21:14.180 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: SELECT ?g ?s ?p ?o { GRAPH ?g { ?s ?p ?o} }
15:21:14.183 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  g=http://example.com/tutorial/description/una1# s=http://example.com/tutorial/description/una1#C4.I1 p=http://opencaesar.io/oml#hasSource o=http://example.com/tutorial/description/una1#I1
15:21:14.183 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  g=http://example.com/tutorial/description/una1# s=http://example.com/tutorial/description/una1#C4.I1 p=http://opencaesar.io/oml#hasSource o=http://example.com/tutorial/description/una1#C4
15:21:14.183 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  g=http://example.com/tutorial/description/una1# s=http://example.com/tutorial/description/una1#C4.I1 p=http://www.w3.org/1999/02/22-rdf-syntax-ns#type o=http://imce.jpl.nasa.gov/foundation/mission#Presents
15:21:14.183 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  g=http://example.com/tutorial/description/una1# s=http://example.com/tutorial/description/una1#C4 p=http://www.w3.org/1999/02/22-rdf-syntax-ns#type o=http://imce.jpl.nasa.gov/foundation/mission#Component
15:21:14.183 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (4 results)
```

The union graph query produces 214 triples, the same as before the sparql update.

At the API level, how do we make the reasoner run again?
If we were to use a Fuseki configuration, is there a way to make sure that the reasoner will run after every SPARQL update?

```text
15:21:14.183 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - after insertion ds1: Check how many results we get querying the union graph.
15:21:14.183 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: SELECT * {?s ?p ?o}
15:21:14.188 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (214 results)
```

```text
15:21:14.188 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - after insertion ds1: Check named graphs for patterns: ?x mission:presents ?y.
15:21:14.188 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: PREFIX mission: <http://example.com/tutorial/vocabulary/mission#>
SELECT ?g ?c ?i WHERE { GRAPH ?g { ?c mission:presents ?i . } }
ORDER BY ?g ?c ?i
15:21:14.190 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (0 results)
```

The following query produces the same results as before the sparql update; further indication that the reasoner has not run.

```text
15:21:14.190 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - after insertion ds1: Check union graph for patterns: ?x mission:presents ?y.
15:21:14.190 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: PREFIX mission: <http://example.com/tutorial/vocabulary/mission#>
SELECT ?c ?i WHERE { ?c mission:presents ?i . }
ORDER BY ?c ?i
15:21:14.193 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  c=http://example.com/tutorial/description/una1#C1 i=http://example.com/tutorial/description/una1#I1
15:21:14.193 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  c=http://example.com/tutorial/description/una1#C2 i=http://example.com/tutorial/description/una1#I2
15:21:14.193 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  c=http://example.com/tutorial/description/una1#C3 i=http://example.com/tutorial/description/una1#I3
15:21:14.193 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (3 results)
```


The following query produces the same results as before the sparql update; further indication that the reasoner has not run.

```text
15:21:14.193 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - after insertion ds1: Check union graph for patterns: ?x a mission:Component; ?x a ?t.
15:21:14.193 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX mission:     <http://example.com/tutorial/vocabulary/mission#>SELECT * {?s a mission:Component; a ?t }
15:21:14.198 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C2 t=http://example.com/tutorial/vocabulary/mission#IdentifiedThing
15:21:14.199 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C2 t=http://example.com/tutorial/vocabulary/mission#Component
15:21:14.199 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C2 t=http://example.com/tutorial/vocabulary/mission#RadHardComponent
15:21:14.199 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C2 t=http://www.w3.org/2002/07/owl#NamedIndividual
15:21:14.199 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C3 t=http://example.com/tutorial/vocabulary/mission#IdentifiedThing
15:21:14.199 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C3 t=http://example.com/tutorial/vocabulary/mission#Component
15:21:14.199 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C3 t=http://www.w3.org/2002/07/owl#NamedIndividual
15:21:14.199 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C1 t=http://example.com/tutorial/vocabulary/mission#IdentifiedThing
15:21:14.200 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C1 t=http://example.com/tutorial/vocabulary/mission#Component
15:21:14.200 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C1 t=http://www.w3.org/2002/07/owl#NamedIndividual
15:21:14.200 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (10 results)
15:21:14.200 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - after insertion ds1: Check named graphs for patterns: ?x a mission:Component; ?x a ?t.
15:21:14.200 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX mission:     <http://example.com/tutorial/vocabulary/mission#>SELECT * { GRAPH ?g { ?s a mission:Component; a ?t } }
15:21:14.203 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (0 results)
```

No change to statement counts; again, further indication that the reasoner has not run.

```text
15:21:14.203 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - statements (base) = 182
15:21:14.204 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - statements (inf)  = 208
```

## 9) Testing sparql delete update

The deletion did not remove the 4 triples we inserted.

What is the proper way to perform delete updates and trigger reasoning?

```text
15:21:14.205 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - DELETE...
15:21:14.223 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - after deletion ds1: Check how many results we get querying named graphs.
15:21:14.223 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: SELECT ?g ?s ?p ?o { GRAPH ?g { ?s ?p ?o} }
15:21:14.224 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (4 results)
```

```text
15:21:14.225 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - after deletion ds1: Check how many results we get querying the union graph.
15:21:14.225 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: SELECT * {?s ?p ?o}
15:21:14.227 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (214 results)
15:21:14.227 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - after deletion ds1: Check named graphs for patterns: ?x mission:presents ?y.
15:21:14.227 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: PREFIX mission: <http://example.com/tutorial/vocabulary/mission#>
SELECT ?g ?c ?i WHERE { GRAPH ?g { ?c mission:presents ?i . } }
ORDER BY ?g ?c ?i
15:21:14.228 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (0 results)
15:21:14.228 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - after deletion ds1: Check union graph for patterns: ?x mission:presents ?y.
15:21:14.228 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: PREFIX mission: <http://example.com/tutorial/vocabulary/mission#>
SELECT ?c ?i WHERE { ?c mission:presents ?i . }
ORDER BY ?c ?i
15:21:14.229 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  c=http://example.com/tutorial/description/una1#C1 i=http://example.com/tutorial/description/una1#I1
15:21:14.229 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  c=http://example.com/tutorial/description/una1#C2 i=http://example.com/tutorial/description/una1#I2
15:21:14.229 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  c=http://example.com/tutorial/description/una1#C3 i=http://example.com/tutorial/description/una1#I3
15:21:14.229 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (3 results)
15:21:14.229 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - after deletion ds1: Check union graph for patterns: ?x a mission:Component; ?x a ?t.
15:21:14.229 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX mission:     <http://example.com/tutorial/vocabulary/mission#>SELECT * {?s a mission:Component; a ?t }
15:21:14.230 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C2 t=http://example.com/tutorial/vocabulary/mission#IdentifiedThing
15:21:14.230 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C2 t=http://example.com/tutorial/vocabulary/mission#Component
15:21:14.230 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C2 t=http://example.com/tutorial/vocabulary/mission#RadHardComponent
15:21:14.230 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C2 t=http://www.w3.org/2002/07/owl#NamedIndividual
15:21:14.231 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C3 t=http://example.com/tutorial/vocabulary/mission#IdentifiedThing
15:21:14.231 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C3 t=http://example.com/tutorial/vocabulary/mission#Component
15:21:14.231 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C3 t=http://www.w3.org/2002/07/owl#NamedIndividual
15:21:14.231 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C1 t=http://example.com/tutorial/vocabulary/mission#IdentifiedThing
15:21:14.231 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C1 t=http://example.com/tutorial/vocabulary/mission#Component
15:21:14.231 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d -  s=http://example.com/tutorial/description/una1#C1 t=http://www.w3.org/2002/07/owl#NamedIndividual
15:21:14.231 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (10 results)
15:21:14.231 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - after deletion ds1: Check named graphs for patterns: ?x a mission:Component; ?x a ?t.
15:21:14.231 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - <<< query: PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX mission:     <http://example.com/tutorial/vocabulary/mission#>SELECT * { GRAPH ?g { ?s a mission:Component; a ?t } }
15:21:14.233 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - >>> query (0 results)
15:21:14.233 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - statements (base) = 182
15:21:14.233 [main] INFO  io.opencaesar.owl.fuseki_reasoner.OwlReasonIncrementallyJena1d - statements (inf)  = 208

```