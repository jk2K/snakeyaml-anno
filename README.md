[![Build Status](https://travis-ci.org/beosign/snakeyaml-anno.svg?branch=development)](https://travis-ci.org/beosign/snakeyaml-anno)

[![codecov](https://codecov.io/gh/beosign/snakeyaml-anno/branch/development/graph/badge.svg)](https://codecov.io/gh/beosign/snakeyaml-anno?branch=development)


# snakeyaml-anno
Parse YAML files by using annotation in POJOS - based on SnakeYaml **1.23** by Sergey Pariev, https://github.com/spariev/snakeyaml/.

## Compatibility

<table class="tg">
  <tr>
    <th class="tg-us36">SnakeyamlAnno</th>
    <th class="tg-us36" colspan="7">SnakeYaml</th>
  </tr>
  <tr>
    <td class="tg-us36"></td>
    <td class="tg-us36">1.17</td>
    <td class="tg-us36">1.18</td>
    <td class="tg-us36">1.19</td>
    <td class="tg-us36">1.20</td>
    <td class="tg-us36">1.21</td>
    <td class="tg-us36">1.22</td>
    <td class="tg-us36">1.23</td>
  </tr>
  <tr>
    <td class="tg-us36">0.3.0<br></td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">NO</td>
    <td class="tg-us36">NO</td>
    <td class="tg-us36">NO</td>
    <td class="tg-us36">NO</td>
    <td class="tg-us36">NO</td>
  </tr>
  <tr>
    <td class="tg-us36">0.4.0</td>
    <td class="tg-us36">NO<br></td>
    <td class="tg-us36">NO<br></td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
  </tr>
  <tr>
    <td class="tg-us36">0.5.0</td>
    <td class="tg-us36">NO<br></td>
    <td class="tg-us36">NO<br></td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
  </tr>
  <tr>
    <td class="tg-us36">0.6.0</td>
    <td class="tg-us36">NO<br></td>
    <td class="tg-us36">NO<br></td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
  </tr>
  <tr>
    <td class="tg-us36">0.7.0</td>
    <td class="tg-us36">NO<br></td>
    <td class="tg-us36">NO<br></td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
    <td class="tg-us36">YES</td>
  </tr>
  <tr>
    <td class="tg-us36">0.8.0</td>
    <td class="tg-us36">NO<br></td>
    <td class="tg-us36">NO<br></td>
    <td class="tg-us36">?</td>
    <td class="tg-us36">?</td>
    <td class="tg-us36">?</td>
    <td class="tg-us36">?</td>
    <td class="tg-us36">?</td>
  </tr>
</table>

## Usage
You must use the `AnnotationAwareConstructor` when parsing:

```java
Yaml yaml = new Yaml(new AnnotationAwareConstructor(MyRoot.class));
```

You must use the `AnnotationAwareRepresenter` when dumping:

```java
Yaml yaml = new Yaml(new AnnotationAwareRepresenter());
```

## Features

### Quick Overview

* Property name mapping
* Converting
* Custom Constructor
* Case insensitive parsing
* Allow parsing of single value for Collection property
* Allow parsing of list at root without tags
* Ignore parsing errors in a subtree
* Auto type detection
* Skipping properties
* Ordering properties

### Feature Details

This section covers the details of the feature including examples how to use them. 

#### Property name mapping
If the properties of your POJO and in your yaml do not match regarding the names, you can supply a mapping by using the `Property` annotation.

Suppose you have the following yaml:

```javascript
- name: Homer
  lastname: Simpson
- name: Marge
  lastname: Simpson
```

And this is your POJO

```java
public class Person {
    private String firstName;
    private String lastName;
}
```

In order to be able to parse the yaml file, you can do the following:

```java
public class Person {
    @Property(key = "name")
    private String firstName;
    private String lastName;
}
```
   
It is also possible to annotate the getter instead:

```java 
 public class Person {     
    private String firstName;
    private String lastName;
  
    @Property(key = "name")
    public String getFirstName() {...}
}
```


#### Converting
You can apply a converter to a field or getter. This feature is especially useful for converting enum values as Snakeyaml as of version 1.17 supports only basic converting, e.g. the string in the yaml file must match the enum constant definition in Java (uppercase).
The following example shows how to convert `m` to the enum constant `MALE`:

```javascript
- name: Homer
  gender: m
- name: Marge
  gender: f
```

```java
public enum Gender {
    MALE("m"),
    FEMALE("f");

    private String abbr;

    private Gender(String abbr) {
        this.abbr = abbr;
    }

    public String getAbbr() {
        return abbr;
    }
}
```

```java 
 public class Person {     
    private String name;
   
    @Property(converter=GenderConverter.class)
    private Gender gender;
}
```

```java 
public class GenderConverter implements Converter<Gender> {

    @Override
    public String convertToYaml(Gender modelValue) {
        return modelValue.getAbbr();
    }

    @Override
    public Gender convertToModel(String value) {
        for (Gender g : Gender.values()) {
            if (g.getAbbr().equals(value)) {
                 return g;
            }
        }
        
        return null;
    }
}
```

As of version 0.4.0, conversion is also implemented for dumping. The interface has changed; the `convertToModel` method now takes a `String` as parameter instead of `Node`.

#### Custom Constructor
The converter example above has shown how to apply a different logic to parse a node into a Java object.

However, imagine that a property of type `Gender` would be more than once. When using a converter, you would have to annotate each property of type `Gender`.

So it could make sense to tell the parser to apply a custom logic when it encounters a property of a certain type like `Gender`. This is where the concept of a _CustomConstructor_ comes in handy.

##### Register Custom Constructor by class annotation 
Given the same classes as in the converter example, instead of defining a `Converter` and annotating each property of type `Gender` with it, you can 
annotate the enum `Gender` with the `ConstructBy` annotation:


```java
@ConstructBy(GenderCustomConstructor.class)
public enum Gender {
    MALE("m"),
    FEMALE("f");

    private String abbr;

    private Gender(String abbr) {
        this.abbr = abbr;
    }

    public String getAbbr() {
        return abbr;
    }
}
```

This instructs the parser to create a Java object from a node of type `Gender` using the given `GenderCustomConverter` class, which must implement the `CustomConstructor<T>` interface and could be defined as follows:

```java
public class GenderCustomConstructor implements CustomConstructor<Gender> {
    @Override
    public Gender construct(Node node, Function<? super Node, ? extends Gender> defaultConstructor) throws ConstructorException {
        String val = (String) NodeUtil.getValue(node); // contains 'm' or 'f'
    
        for (Gender g : Gender.values()) {
           if (g.getAbbr().equals(value)) {
              return g;
           }
        }
        
        // try default way of parsing the enum
        return defaultConstructor.apply(node); // if string contains "MALE" or "FEMALE", this is also ok
}

```

The custom constructor can make use of the default way of constructing the passed in node by using the passed in `defaultConstructor` instance.

##### Register Custom Constructor by code
It may be the case that you want to parse a yaml node into a Java object whose type is not defined by your application, but instead comes from a thrid party library or Java itself, so it is not possible to put an annotation on that class.

In this case, you can register a custom constructor using `AnnotationAwareConstructor.registerCustomConstructor`:

```java
   annotationAwareConstructor.registerCustomConstructor(Gender.class, GenderCustomConstructor.class);
   Yaml yaml = new Yaml(annotationAwareConstructor);
```

For more control over the registered constructors, you can also modify `AnnotationAwareConstructor.getConstructByMap`.

##### Custom Constructor Inheritance
If a custom constructor is registered for a type `S`, and a node is of type `T` with `T extends/implements S`, then the custom constructor will also be used. So a custom constructor on a type will also be used for any subtypes.

For example, if a custom constructor is registered for type `Number`, then it will be called for properties of type `Number`, but also for properties of type `Integer` and `Double`. If for let's say `Integer` the custom converter should not be applied, then one has to register the `DefaultCustomConstructor`:  `annotationAwareConstructor.registerCustomConstructor(Integer.class, DefaultCustomConstructor.class)`.

##### Custom Constructor Resolution
Because a custom constructor can be registered via annotation or programmatically, it is possible that there are two definitions for a given (super-)type.

The type hierarchy is relevant. Given a node of type `T`, the exact way for finding the `ConstructBy` instance (and thus the custom constructor class) is as follows:

1. Start with `T`, then walk the superclass hierarchy of `T`, then all interfaces of `T`.
1. For each class/interface `S super T`, check first if there is an entry in the `getConstructByMap()` for `S` and if so, return the `ConstructBy` from the map
1. Check if `S` is annotated with `ConstructBy`, and if so, return the `ConstructBy` from the annotation.
1. If there is no match for `S`, proceed with the next class/interface in the hierarchy
1. If no match was found after walking the whole class hierarchy of `T`, no custom constructor is used.

##### Register Custom Constructor on property

It is also possible to register a custom constructor on a per-property-basis. Use this instead of a `Converter` if you need more than a simple  conversion mechanism. 

#### Case insensitive parsing
A flag can be passed so that parsing is possible even if the keys in the yaml file do not match the case of the java property where it sould be parsed into. To enable it, use `AnnotationAwareConstructor constructor = new AnnotationAwareConstructor(Person.class, true)`.
So for example, all of the following variants can be parsed using the same Person class (see above):

```javascript
  Name: Homer
  nAME: Marge
  NaMe: Bart
  name: Lisa
```

In the very unlikely case that a Java Bean class contains two properties that differ only in case, the result which property is used is undetermined.

#### Allow parsing of single value for Collection property
If you have a collection based property, you have to provide a list in SnakeYaml, otherwise the value cannot be parsed. Example:

```java 
 public class Person {     
    private String name;
   
    private List<Integer> favoriteNumbers;
    private List<Person> children;

}
```

The corresponding parseable yaml could look like this:

```javascript
name: Homer
favoriteNumbers: [42]
children: [{name: bart}]
```

Even though there is only one favorite number, you have to provide the "42" as list item. With snakeyaml-anno, it is possible to provide the following yaml - the constructed nodes will be automatically put into a singleton list, letting the parse process succeed.

```javascript
name: Homer
favoriteNumbers: 42
children: {name: bart}
```
#### Allow parsing of list at root without tags
Currently, there is no simple way to parse the following yaml, where the yaml consists of a root list and the type of a list item is **not** provided by adding a tag to each list item:

```
- id: 1
  name: One
- id: 2
  name: Two
```

Instead, you need to use tags:

```
- !MyClass 
  id: 1
  name: One
- !MyClass 
  id: 2
  name: Two
```

In order to be able to parse the first example, you can use the following:

```
// Parse a list where each list item is of type Person
annotationAwareConstructor = new AnnotationAwareConstructor(List.class, Person.class, false);
Yaml yaml = new Yaml(annotationAwareConstructor);
```

#### Ignore parsing errors
In a complex hierarchy it may be desirable to ignore parse errors in a given subtree and still return the parsed objects higher up the tree. In case of an exception, the unparsable object will simply remain `null`. To allow the parsing process to skip unparsable parts instead of aborting, you can use `ignoreExceptions = true` on a property or a getter:

```java 
 public class Person {     
    private String firstName;
    private String lastName;
    
    @Property(ignoreExceptions = true)
    private Gender gender;
  
}
```

So in case the gender property cannot be parsed, you still get a parsed Person object, just with the gender property being `null`.

#### Auto type detection
YAML uses the concept of _Tags_ to provide type information. However, to keep the YAML file as simple and concise as possible, it may be desirable to omit a tag declaration if the concrete type to use can already be deducted from the properties. Suppose you have the following interface:

```java 
public interface Animal {     
    String getName();  
}
```

And two implementations:

```java 
public class Dog implements Animal {     
   private int nrOfBarksPerDay;  
   ...
}
```

```java 
public class Cat implements Animal {     
   private int miceCaughtCounter;  
   ...
}
```

And the container:

```java 
public class Person {     
   private List<Animal> pets;  
   ...
}
```

For the following YAML, the first object in the list must be of type `Dog` and the second of type `Cat`.

```javascript
pets:
- name: Santas Little Helper
  nrOfBarksPerDay: 20
- name: Snowball
  miceCaughtCounter: 20
```

You have two ways of telling YAML to autodetect the types: annotation based or programmatic. In either way, use must provide possible subtypes for a given supertype because classpath scanning of classes implementing a given interface is not yet implemented.
If no valid substitution class if found, the default SnakeYaml algorithm for choosing the type will be used. If multiple substitution types are possible, the first possible type (determined by the order of the classes in `substitutionTypes`) is chosen. You can also provide your own `SubstitutionTypeSelector` implementation and set it with `@Type(substitutionTypes = { Dog.class, Cat.class }, substitutionTypeSelector = MyTypeSelector.class)`.  `MyTypeSelector` must implement `SubstitutionTypeSelector` and must have a no-arg constructor.

##### Annotation-based
To tell YAML to autodetect the types, you have to annotate the `Animal` interface with `@Type(substitutionTypes = { Dog.class, Cat.class })`. Optionally, you can register a `SubstitutionTypeSelector`.

##### Programmatic
Before loading, you can register a mapping from an interface to `@Type` annotation instance. Optionally, you can modify and/or remove any mappings that have already been determined by evaluating the annotations.

```java
 AnnotationAwareConstructor constructor = new AnnotationAwareConstructor(BaseClassOrInterface.class);

// optionally override given entries that have been inserted by evaluating annotations
constructor.getTypesMap().clear();

// register type programmatically
// this has the same effect as putting an annotation of the form @Type(substitutionTypes = {Concrete1.class, Concrete2.class} on BaseClassOrInterface class
constructor.getTypesMap().put(BaseClassOrInterface.class, new TypeImpl(new Class<?>[] { Concrete1.class, Concrete2.class }));

Yaml yaml = new Yaml(constructor);
```

#### Skipping properties

##### Skip dumping of empty properties globally
By default, empty / null properties (see remarks for `SkipIfNull`and `SkipfIfEmpty` below) will be skipped during dumping. You can change that by

```java
Yaml yaml = new Yaml(new AnnotationAwareRepresenter(false)); // supply "false" flag
```

When deactivating the global skipping of empty properties, one can determine on a per-property-basis if a property is to be dumped.

**If global skipping of empty properties is activated (by default), then `skipAtDump` and `skipAtDumpIf` will have no effect**.

##### Skip properties locally

It is possible to skip properties during load or dump. In order to skip a property during load, thus preventing snakeyaml to override a model value with the value read from the yaml file that is being loaded, annotate the property with `skipAtLoad`:

```java 
public class Person {

   @Property(skipAtLoad = true)     
   private String name;
}
```

In order to prevent dumping of a property, use `skipAtDump`:
 
```java 
public class Person {

   @Property(skipAtDump = true)     
   private String name;
}
```

You can also skip dumping conditionally by implementing the `SkipAtDumpPredicate` interface. The only method to implement is `skip`. One of the parameters passed into this method is the property value, so you can make decisions whether to skip a property based on its value.
You can use your implementation by using the `skipAtDumpIf` member:
 
```java 
public class Person {

   @Property(skipAtDumpIf = SkipIfNull.class)     
   private String name;
}
```

**Be aware** that if `skipAtDump` is also supplied and set to true, it will take precedence over the ` skipAtDumpIf`! `skipAtDump` and `skipAtDumpIf` will have **no effect** if empty properties are skipped globally (that's the default, see above).

Predefined are two classes: `SkipIfNull` and `SkipIfEmpty`. The first one skips a property if it is `null`, the latter one skips a property if it is of type `Map`, `Collection` or `String` and the property is empty (empty map/collection or String of length 0).


#### Ordering properties
It is possible to order properties during the dump process by providing a value for the `order` property:

```java 
public class Person {

   @Property(order = 5)     
   private String first;

     
   private String between;

   @Property(order = -5)     
   private String last;
}
```

This will dump the properties in the order:

1. first
2. between
3. last

The default value for `order` is 0. A higher value means that the property is dumped before a property with a lower value. So in order to dump a property at the beginning, you can provide a positive value. To make sure that a property is dumped at the end, you can provide a negative value. The order of properties with the same `order` value is unspecified.

 