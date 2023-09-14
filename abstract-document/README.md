---
title: Abstract Document
category: Structural
language: en
tag: 
 - Extensibility
---

## Intent

The "Abstract Document" design pattern is a structural design pattern that aims to provide a consistent way to handle 
hierarchical and tree-like data structures by defining a common interface for various document types.

## Explanation

The Abstract Document pattern enables handling additional, non-static properties. This pattern
uses concept of traits to enable type safety and separate properties of different classes into
set of interfaces.

Real world example

>  Consider a car that consists of multiple parts. However, we don't know if the specific car really has all the parts, or just some of them. Our cars are dynamic and extremely flexible.

In plain words

> Abstract Document pattern allows attaching properties to objects without them knowing about it.

Wikipedia says

> An object-oriented structural design pattern for organizing objects in loosely typed key-value stores and exposing 
the data using typed views. The purpose of the pattern is to achieve a high degree of flexibility between components 
in a strongly typed language where new properties can be added to the object-tree on the fly, without losing the 
support of type-safety. The pattern makes use of traits to separate different properties of a class into different 
interfaces.

**Programmatic Example**

Let's first define the base classes `Document` and `AbstractDocument`. They basically make the object hold a property
map and any amount of child objects.

```java
public interface Document {

  Void put(String key, Object value);

  Object get(String key);

  <T> Stream<T> children(String key, Function<Map<String, Object>, T> constructor);
}

public abstract class AbstractDocument implements Document {

  private final Map<String, Object> properties;

  protected AbstractDocument(Map<String, Object> properties) {
    Objects.requireNonNull(properties, "properties map is required");
    this.properties = properties;
  }

  @Override
  public Void put(String key, Object value) {
    properties.put(key, value);
    return null;
  }

  @Override
  public Object get(String key) {
    return properties.get(key);
  }

  @Override
  public <T> Stream<T> children(String key, Function<Map<String, Object>, T> constructor) {
    return Stream.ofNullable(get(key))
        .filter(Objects::nonNull)
        .map(el -> (List<Map<String, Object>>) el)
        .findAny()
        .stream()
        .flatMap(Collection::stream)
        .map(constructor);
  }
  ...
}
```
Next we define an enum `Property` and a set of interfaces for type, price, model and parts. This allows us to create
static looking interface to our `Car` class.

```java
public enum Property {

  PARTS, TYPE, PRICE, MODEL
}

public interface HasType extends Document {

  default Optional<String> getType() {
    return Optional.ofNullable((String) get(Property.TYPE.toString()));
  }
}

public interface HasPrice extends Document {

  default Optional<Number> getPrice() {
    return Optional.ofNullable((Number) get(Property.PRICE.toString()));
  }
}
public interface HasModel extends Document {

  default Optional<String> getModel() {
    return Optional.ofNullable((String) get(Property.MODEL.toString()));
  }
}

public interface HasParts extends Document {

  default Stream<Part> getParts() {
    return children(Property.PARTS.toString(), Part::new);
  }
}
```

Now we are ready to introduce the `Car`.

```java
public class Car extends AbstractDocument implements HasModel, HasPrice, HasParts {

  public Car(Map<String, Object> properties) {
    super(properties);
  }
}
```

And finally here's how we construct and use the `Car` in a full example.

```java
    LOGGER.info("Constructing parts and car");

    var wheelProperties = Map.of(
        Property.TYPE.toString(), "wheel",
        Property.MODEL.toString(), "15C",
        Property.PRICE.toString(), 100L);

    var doorProperties = Map.of(
        Property.TYPE.toString(), "door",
        Property.MODEL.toString(), "Lambo",
        Property.PRICE.toString(), 300L);

    var carProperties = Map.of(
        Property.MODEL.toString(), "300SL",
        Property.PRICE.toString(), 10000L,
        Property.PARTS.toString(), List.of(wheelProperties, doorProperties));

    var car = new Car(carProperties);

    LOGGER.info("Here is our car:");
    LOGGER.info("-> model: {}", car.getModel().orElseThrow());
    LOGGER.info("-> price: {}", car.getPrice().orElseThrow());
    LOGGER.info("-> parts: ");
    car.getParts().forEach(p -> LOGGER.info("\t{}/{}/{}",
        p.getType().orElse(null),
        p.getModel().orElse(null),
        p.getPrice().orElse(null))
    );

    // Constructing parts and car
    // Here is our car:
    // model: 300SL
    // price: 10000
    // parts: 
    // wheel/15C/100
    // door/Lambo/300
```

## Class diagram

![alt text](./etc/abstract-document.png "Abstract Document Traits and Domain")

## Applicability

This pattern is particularly useful in scenarios where you have different types of documents that share some common 
attributes or behaviors, but also have unique attributes or behaviors specific to their individual types. Here are 
some scenarios where the Abstract Document design pattern can be applicable:

* Content Management Systems (CMS): In a CMS, you might have various types of content such as articles, images, videos, etc. Each type of content could have shared attributes like creation date, author, and tags, while also having specific attributes like image dimensions for images or video duration for videos.

* File Systems: If you're designing a file system where different types of files need to be managed, such as documents, images, audio files, and directories, the Abstract Document pattern can help provide a consistent way to access attributes like file size, creation date, etc., while allowing for specific attributes like image resolution or audio duration.

* E-commerce Systems: An e-commerce platform might have different product types such as physical products, digital downloads, and subscriptions. Each type could share common attributes like name, price, and description, while having unique attributes like shipping weight for physical products or download link for digital products.

* Medical Records Systems: In healthcare, patient records might include various types of data such as demographics, medical history, test results, and prescriptions. The Abstract Document pattern can help manage shared attributes like patient ID and date of birth, while accommodating specialized attributes like test results or prescribed medications.

* Configuration Management: When dealing with configuration settings for software applications, there can be different types of configuration elements, each with its own set of attributes. The Abstract Document pattern can be used to manage these configuration elements while ensuring a consistent way to access and manipulate their attributes.

* Educational Platforms: Educational systems might have various types of learning materials such as text-based content, videos, quizzes, and assignments. Common attributes like title, author, and publication date can be shared, while unique attributes like video duration or assignment due dates can be specific to each type.

* Project Management Tools: In project management applications, you could have different types of tasks like to-do items, milestones, and issues. The Abstract Document pattern could be used to handle general attributes like task name and assignee, while allowing for specific attributes like milestone date or issue priority.

The key idea behind the Abstract Document design pattern is to provide a flexible and extensible way to manage different 
types of documents or entities with shared and distinct attributes. By defining a common interface and implementing it 
across various document types, you can achieve a more organized and consistent approach to handling complex data 
structures.

## Credits

* [Wikipedia: Abstract Document Pattern](https://en.wikipedia.org/wiki/Abstract_Document_Pattern)
* [Martin Fowler: Dealing with properties](http://martinfowler.com/apsupp/properties.pdf)
* [Pattern-Oriented Software Architecture Volume 4: A Pattern Language for Distributed Computing (v. 4)](https://www.amazon.com/gp/product/0470059028/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=javadesignpat-20&creative=9325&linkCode=as2&creativeASIN=0470059028&linkId=e3aacaea7017258acf184f9f3283b492)
