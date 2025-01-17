# How to work with ontologies

OSP-core supports ontologies in the following formats:
- [OWL ontologies](working_with_ontologies.md#owl-ontologies-and-rdfs-vocabularies)
- [RDFS vocabularies](working_with_ontologies.md#owl-ontologies-and-rdfs-vocabularies) (_[limited support](#rdfs-vocabularies)_)
- [OSP-core YAML ontology format](working_with_ontologies.md#osp-core-yaml-ontology-format)

## OWL ontologies and RDFS vocabularies

To install OWL ontologies or RDFS vocabularies in OSP-core, you have to 
create a configuration yaml file similar to the following one:

```yaml
identifier: emmo
ontology_file: https://raw.githubusercontent.com/emmo-repo/EMMO/master/emmo-inferred.owl
format: turtle
reference_by_label: True
namespaces:
    mereotopology: http://emmo.info/emmo/top/mereotopology
    physical: http://emmo.info/emmo/top/physical
    top: http://emmo.info/emmo/top
    semiotics: http://emmo.info/emmo/top/semiotics
    perceptual: http://emmo.info/emmo/middle/perceptual
    reductionistic: http://emmo.info/emmo/middle/reductionistic
    holistic: http://emmo.info/emmo/middle/holistic
    physicalistic: http://emmo.info/emmo/middle/physicalistic
    math: http://emmo.info/emmo/middle/math
    properties: http://emmo.info/emmo/middle/properties
    materials: http://emmo.info/emmo/middle/materials
    metrology: http://emmo.info/emmo/middle/metrology
    models: http://emmo.info/emmo/middle/models
    manufacturing: http://emmo.info/emmo/middle/manufacturing
    isq: http://emmo.info/emmo/middle/isq
    siunits: http://emmo.info/emmo/middle/siunits
active_relationships:
    - http://emmo.info/emmo/top/mereotopology#EMMO_8c898653_1118_4682_9bbf_6cc334d16a99
    - http://emmo.info/emmo/top/semiotics#EMMO_60577dea_9019_4537_ac41_80b0fb563d41
default_relationship: http://emmo.info/emmo/top/mereotopology#EMMO_17e27c22_37e1_468c_9dd7_95e137f73e7f
```

### Keywords

**identifier**: Can be any alphanumerical string. It is the name of the package
that contains multiple namespaces. Will be used for uninstallation: `pico uninstall emmo`.
(In YAML ontologies this package name or identifier is the same as the namespace name).

**ontology_file**: Path to the inferred owl ontology. That means you should
have executed a reasoner on your ontology, e.g. by using the `Export inferred axioms`
functionality of [Protégé](https://protege.stanford.edu/).

**format** (optional): File format of the ontology file to be parsed. We 
support all the
formats that
[RDFLib](https://rdflib.readthedocs.io/en/stable/plugin_parsers.html) supports:
XML (`xml`, `application/rdf+xml`, default), Turtle (`turtle`, `ttl`, 
`text/turtle`), N3 (`n3`,`text/n3`), NTriples (`nt`, `nt11`, 
`application/n-triples`), N-Quads (`nquads`, `application/n-quads`), 
TriX (`trix`, `application/trix`) and TriG (`trig`, `application/trig`). 
When not provided, it will be guessed from the file extension. However, such 
guess may not always be correct.

**reference_by_label** (default False): Whether the label should be used or the IRI suffix to reference
entity from within OSP-core. In case of EMMO it is true, because IRI suffixes are not
human friendly. In this case all labels should be unique and not contain whitespaces.
If False, use dot notation to get by IRI square brackets (`__getitem__`) to get by label.
The latter will return a list of all entities with the same label.

**namespaces**: mapping from namespace name (used to import the namespace) to iri prefix.
If IRI doesn't end with "/" or "#", "#" will be added.

**active relationships**:
List of iris of active relationships.

**default relationship**:
The default relationship.

### Installation

Name the yaml file as you would any yaml file `<name>.yml`, where `<name>` should be replaced by a user defined name.

Then you can use pico to install the tool [Pico](utils.md#pico-installs-cuds-ontologies)
to install the ontology:

```sh
pico install </path/to/name.yml>
```

### Limitations

At the moment, there are a few limitations on the supported features of OWL 
ontologies and RDFS vocabularies.

#### OWL ontologies

Not all predicates of OWL ontologies are taken into 
consideration. Among the used ones are:

- `RDF.type` to determine the type of the entities.
- `RDFS.label` to get the entities by label.
- `RDFS.isDefinedBy` to get a descriptions for the entities.
- `RDFS.subClassOf` / `RDFS.subPropertyOf` for subclasses.
- `OWL.inverseOf` for inverse relationships.
- `RDFS.range` to determine the datatype of `DataProperties`. These are the supported
  datatypes:
  - `XSD.boolean`
  - `XSD.integer`
  - `XSD.float`
  - `XSD.string`
- To get the attributes of an owl class, we use
  - The `RDFS.domain` of the `DatatypeProperties`, if it is a simple class.
  - Restrictions on the ontology classes.
  - Furthermore, all DataProperties are considered functional, see [this issue](https://github.com/simphony/osp-core/issues/416).
- Restrictions and compositions are also supported. They can be consulted 
  using the [`axioms` attribute of ontology classes](jupyter/ontology_interface.html#Operations-specific-to-ontology-axioms).

No reasoner is included. We plan to include a reasoner in the
future.

We try to extend this list over time and support more of the
OWL DL standard.

#### RDFS vocabularies

With respect to RDFS vocabularies, the `RDFS.Class` predicate is supported, 
but the `RDFS.Property` predicate is not. This means that the main 
limitation when using RDFS vocabularies is that only their classes are 
detected, but their properties are ignored.

## OSP-core YAML ontology format

This section describes how you can create ontologies using YAML.

```{tip}
   If you have an ontology where all entity names are in ALL_UPPERCASE,
   you can use the commandline tool `yaml2camelcase` that is shipped with
   OSP-core to transform it to an ontology with CamelCase entity names.
```

### Introduction

In this file we will give a description of how an Ontology can be
represented in a yaml file format and how to interpret such files. For
simplicity reasons in the following we will give examples from the
**\* example ontology \*** file which can be found in osp/core/ontology/yml/ontology.city.yml.

### Naming of the files and installation

Name any ontology `<name>.ontology.yml`, where `<name>` should be replaced by a user defined name.

Then you can use pico to install the tool [Pico](#pico-installs-cuds-ontologies)
to install the ontology:

```sh
pico install </path/to/my_ontology.ontology.yml>
```

### Syntax of the .yml ontology

`version`: string

> Contains semantic version Major.minor in format M.m with M and m
> positive integers. minor MUST be incremented when backward
> compatibility in the format is preserved. Major MUST be incremented
> when backward compatibility is removed. Due to the strict nature of
> the format, a change in minor is unlikely.

`namespace`: string

> Defines the namespace of the current file. We recommend to use
  all_lowercase for the namespace name, with underscore as separation.
  All entities defined in this file will live in the namespace defined here.

`requirements`: List[string]

> A list of namespaces that this namespace depends on.

`author`: string

> Reference to the person(s) who created the file.

`ontology`: dict

> Contains individual declarations for ontology entities as a mapping.
> Each key of the mapping is the name of an ontology entity.
> The key can contain letters, numbers and underscore.
> By convention, they should be in CamelCase. The name of ontology classes
> should start with an uppercase latter, while the name of relationships
> and attributes should start with a lower case letter.
> This key is later used the reference the entity from within OSP-core
> in a case sensitive manner.
> The value of the mapping is a mapping whose format is detailed in the
> [Ontology entities format](#ontology-entities-format).

### Ontology entities format

Every declaration of an ontology entity must have the following keys:

`description`: string
> For human consumption. An ontological short description of the carried
> semantics. Should have the form: entity is a superclass\_entity that
> has \<differentiating\> terms.
>
`subclass_of`: List[**\`\`qualified entity name\`\`**].
> Its value is fixed on the ontology level.
>
> The subclass keyword expresses an **ontological is-a**
> relation. MUST be a list of a fully qualified strings referring to another entity.
> Only the entity `cuba.entity` is allowed to have no superclass. See [CUBA namespace](#the-cuba-namespace).
>
> If entity A is a subclass of B and B is subclass of C,
> then A is also subclass of C.

An ontology entity can be either a relationship, a cuds entity or an attribute.
Depending on that the mapping can have further keys.
For cuds entities these keys are described in
[CUDS entities format](#cuds-classes-format) section.
For relationship entities, these keys are described in
[Relationship format](#relationship-format) section.
For attributes, these keys are described in
[Attribute format](#attribute-format) section.

### The CUBA namespace

The CUBA namespace contains a set of Common Universal Basic entities, that have special meaning in OSP-core.
The CUBA namespace is always installed in OSP-core.

`cuba.Entity`
> The root for all ontology classes. Does not have a superclass.

`cuba.Nothing`
> An ontology class, that is not allowed to have individuals.

`cuba.relationship`
> The root of all relationships. Does not have a superclass.

`cuba.attribute`
> The root of all attributes. Does not have a superclass.

`cuba.Wrapper`
> The root of all wrappers. These are the bridge to simulation engines and databases. See the examples in examples/.

`cuba.activeRelationship`
> The root of all active relationships. Active relationships express that one cuds object is in the container of another.

`cuba.passiveRelationship`
> The inverse of `cuba.activeRelationship`.

### Attribute format

Every attribute is a subclass of `cuba.attribute`.
The declaration of an attribute is a special case of the declaration of an entity.
It must have the keys described in [Ontology entities format](#ontology-entities-format).
It can additionally have the following keys:

`datatype`: string

> It is an attribute of an entity in cases when the datatype of said
> entity is important.
>
> Describes the datatype of the value that a certain entity can take. It
> can be one of the following:
>
> - `BOOL`: a form of data with only two possible values (usually
>   \"true\" or \"false\")
> - `INT`: a positive or negative integer number.
> - `FLOAT`: a number containing values on both sides of the decimal
>   point
> - `STRING`: a sequence of characters that can also contain spaces and
>   numbers. The length can be specified with "STRING:LENGTH" (e.g.
>   STRING:20 means the length of the string should be maximum 20
>   characters).
> - `VECTOR:datatype:D1:D2:...:Dn`: a vector of the given dimensions
>   (D1 x D2 x ... x Dn) and the given datatype.
>   The dimensions are always fixed.
>
>   For example, a VECTOR:INT:4:2:1 would be: \
>   { [(a), (b)],  [(c), (d)], [(e), (f)], [(g), (h)] } \
>   where all elements (a, b, ...) are integers.
>   (the different delimiters are only used for visual purposes).
>   If no datatype is specified, it would be a `FLOAT` vector.
>
> In case a datatype is not specified the default datatype is assumed to
> be STRING
>
> For example: The datatype of entity numberOfOccurrences is INT.

> ```{note}
>   The implementation of the vectors is experimental and will be updated as soon as 
>   EMMO has established an appropriate wait of representing them
> ```

### Class expressions

A class expression describes a subset of individuals.
They are similar to classes, but do not have a name in the ontology.
Class expressions will be used in [CUDS entities format](#cuds-classes-format) and [Relationship format](#relationship-format).
They can be either:

- A **\`\`qualified entity name\`\`** of a class. In this case it corresponds to all individuals of referenced class.
- Requirements on the individual's relationships. For example:

  ```yaml
  city.hasInhabitant:
    cardinality: 1+
    range: city.Citizen
    exclusive: false
  ```

  This describes the set of individuals that have at least one citizen as inhabitant.
  In general it describes the individuals which have some relationship to some object.
  It is a mapping from the **\`\`qualified entity name\`\`** of a relationship to the following keywords:

  `range`
  > A class expression describing the individuals which are object of the relationship.

  `cardinality`
  > The number of times individuals defined in `range` is allowed to be a object of the relationship.
  > To define the `cardinality` we use the following syntax:
  >
  > - many / * / 0+ (default): Zero to infinity target objects are allowed.
  > - some / \+ / 1+: At least one target object is required.
  > - ? / 0-1: At most one target object is allowed.
  > - a+: At least a target objects are required.
  > - a-b: At least a and at most b target objects are required (i.e. inclusive).
  > - a: Exactly a target objects are required.

  `exclusive`
  > Whether the given `target` is the only allowed object.
- A composition of several class expressions. For example:

  ```yaml
  or:
    - city.City
    - city.Neighborhood
  ```

  This is the union of all individuals that are a city or a neighbourhood.
  We use the keyword `or` for union, `and` for intersection and `not` for complement.
  After `or` and `and`, a list of  class expressions for the union / intersection is expected.
  After `not` a single class expression is expected.

The definition of class expressions is recursive. For example:

```yaml
or:
  - city.City
  - city.hasPart:
      cardinality: 1+
      range: city.Neighborhood
      exclusive: false
```

This describes the set of all individuals that are a city or have a neighbourhood.

### CUDS classes format

The declaration of a cuds entity is a special case of the declaration of an entity.
It must have the keys described in [Ontology entities format](#ontology-entities-format).
It can contain further information:

`attributes`: Dict[**\`\`qualified entity name\`\`**, default_value]
> Expects a mapping from the **\`\`qualified entity name\`\`** of an attribute to its default.
> Each key must correspond to a subclass of `attribute`. For example:
>
> ```yaml
> Address:
>   ...
>   attributes:
>     city.name: "Street"
>     city.number:
> ```
>
> Here, an address has a name and a number.
> The default name is "Street", the number has no default.
> If no default is given, the user is forced to specify a value each time he creates an individual.

`subclass_of`: List[Class expression]
> In addition to qualified entity names of classes, class expressions are also allowed.
> These class expressions restrict the individuals allowed in the class. Only those
> individuals are allowed that are in the intersection of the class expressions. For example:
>
> ```yaml
> PopulatedPlace:
>    description:
>    subclass_of:
>    - city.GeographicalPlace
>    - city.hasInhabitant:
>        range: city.Citizen
>        cardinality: many
>        exclusive: true
> ```
>
> Here, a populated place is a geographical place which must have citizens as inhabitants.

`disjoint_with`: List[Class expression]
> A list of class expressions that are not allowed to share any individuals with the cuds entity.

`equivalent_to`: List[Class expression]
> A list of class expressions who contain the same individuals as the cuds entity. For example:
>
> ```yaml
> PopulatedPlace:
>    description:
>    equivalent_to:
>    - city.GeographicalPlace
>    - city.hasInhabitant:
>        range: city.Citizen
>        cardinality: many
>        exclusive: true
> ```
>
> Here every geographical place that has citizens as inhabitants is automatically a populated place.

### Relationship format

Every relationship is a subclass of `cuba.relationship`.
The declaration of a relationship is a special case of the declaration of an entity.
It must have the keys described in [Ontology entities format](#ontology-entities-format).
Furthermore, it can contain the following information:

`inverse`: **\`\`qualified entity name\`\`** or empty (None)
> If CUDS object A is related to CUDS object B via relationship `rel`, then B is related
> with A via the inverse of `rel`.
>
> For example: The inverse of `hasPart` is `isPartOf`.
>
> If no inverse is given, OSP-core will automatically create one.

`domain`: Class expression
> A class expression describing the individuals that are allowed to be a subject of the relationship.
> If multiple class expressions are given, the relationship's domain is the intersection of the class expressions.

`range`: Class expression
> A class expression describing the individuals that are allowed to be object of the relationship.
> If multiple class expressions are given, the relationship's range is the intersection of the class expression.

`characteristics`: String
> A list of characteristics of the relationship. The following characteristics are supported:
>
> - reflexive
> - symmetric
> - transitive
> - functional
> - irreflexive
> - asymmetric
> - inversefunctional

A subclass of a relationship is called a sub-relationship.

### Limitations

`Class expressions`, `domain`, `range`, `characteristics`, `equivalent_to`, `disjoint_with`
are currently not parsed by OSP-core.
