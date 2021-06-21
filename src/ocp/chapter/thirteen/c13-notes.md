# Chapter 13 - Annotations

Annotations
- Describe the purpose of annotations and typical usage patterns 
- Apply annotations to classes and methods
- Describe commonly used annotations in the JDK
- Declare custom annotations

## Introducing Annotations (p.554-557)
Annotations are all about METADATA. Metadata is data that provides information about other data. For example, 'attribute data' is the type of data that includes transactional info that makes up the object and its content. On the other hand, 'metadata' includes the rules, properties or relationships surrounding the object. These metadata rules describe information about the object, but are not part of him.

While annotations allow you to insert rules around data, it does not mean the values for these rules need to be defined in the code, aka 'hard coded', you can define the rules and relationships in the code but read the values from elsewhere, such as from a configuration file or database.

### Purpose of Annotations
The purpose of an annotation is to assign metadata attributes to classes, methods, variables and other Java types. Annotations start with the at (@) symbol and can contain attribute/value pairs called 'elements'.

- Example of an annotation: `@ZooAnimal public class Peacock extends Bird { }`

This example brings us to the first rule about annotations: `annotations function a lot like interfaces`. Because in this example annotations allow us to mark a class as a ZooAnimal without changing its inheritance structure.

If they are similar to interfaces, why we just don't use them? While interfaces can be applied only to classes, annotations can be applied to any declaration including classes, methods, expressions and even other annotations. Also, unlike interfaces, annotations allow us to pass a set of values where they are applied. Consider the following example:

    public class Veterinarian {
      @ZooAnimal(habitat="Infirmary") private Lion sickLion;

      @ZooAnimal(habitat="Safari") private Lion healthyLion;

      @ZooAnimal(habitat="Special Enclosure") private Lion blindLion;
    }

The habitat value is part of the type declaration of each variable, not an individual object. This class defines an habitat for each Lion, even if a. Lion variable points to another object, it would still have the same habitat value. This example brings us to the second rule about annotations: `annotations establish relationships that make it easier to manage data about our application.` The third rule about annotations is: `an annotation ascribes custom information on the declaration where it is defined.`. The same annotation can often be applied to completely unrelated classes or variables. There is one final rule about annotations you should be familiar with: `annotations are optional metadata and by themselves do not do anything.`. This means that you can take a project filled with thousand of annotations and remove all of them, and it will still compile and run, albeit with potetially different behavior at runtime.

This last rule might seem a little counterintuitive at first, but it refers to the fact that annotations aren't utilized where they are defined. It's up to the rest of the application, or more likely the underlying framework, to enforce or use annotations to accomplish tasks.

While an annotation can be removed from a class and it will still compile, the opposite is not true; adding an annotation can trigger a compiler error. As we will see in this chapter, the compiler validates that annotations are properly used and include all required fields, like @FunctionalClass or @SafeVarargs.

Annotations, just like methods or fields, can be inherited between class hierarchies. The annotation can be overridden in case the child class has the annotation. Because there is no multiple inheritance in Java, annotations used on interfaces cannot be inherited.

* Note for the exam: You need to know how to define your own custom annotations, how to apply annotations properly and how to use common annotations. Writing code 
that processes or enforces annotations is not required for the exam.

## Creating Custom Annotations (p.558-562)

We use the `@interface` annotation (all lowercase) to declare an annotation. Yes, we use an annotation to create an annotation. Like classes and interfaces, they are commonly defined in their own file as a top-level type, although they can be defined inside a class declaration like an inner class.
- Example of an interface declaration: `public @interface Exercise { }`

This annotation example defined above, is referred as a 'marker annotation', since it does not contain any elements. To apply this interface, you just need to use the at symbol (@):
	
	@Exercise() public class Cheetah { }
	
	@Exercise
	public class ZooEmployee { }
	
When using a marker annotation, the parentheses are optional, because they don't contain any values. If an annotation is declared on a line by itself, then it applies to the next non-annotation type found on the proceeding lines. This applies even when there are multiple annotations present. For example:
    
    @Scaley       @Flexible
      @Food("insect") @Food("rodent")     @FriendlyPet
    @Limbless public class Snake { }
    
Regardless of where they are, all of the annotations apply to Snake. As with other declarations in Java, spaces and tabs between elements are ignored. Whether you put annotations on the same line as the type they apply to or on separate lines is a matter of style, either is acceptable. Annotations are case sensitive, like classes and interfaces names, it is a common practice to have them start with an uppercase letter, although is not required. There are some annotations that can be applied more than once, like @Food, we will cover them later in this chapter.

### Specifying a Required Element

An annotation element is an attribute that stores values about the particular usage of an annotation. For example, lets make @Exercise more useful:
    
    public @interface Exercise {
      int hoursPerDay();
    }
    
hoursPerDay may look a lot like an abstract method, although we're calling it an element (or attribute). Remember, annotations have their roots in interfaces, so behind the scenes, the JVM is creating elements as interface methods and annotations as implementations of these interfaces, you don't need to worry about this details for the exam. The compiler does it all for you too. Adding elements on an annotation changes our usage:
    
    @Exercise(hoursPerDay=3) public class Cheetah { } // COMPILES

    @Exercise hoursPerDay=3 public class Cheetah { } // DOES NOT COMPILE. Remember, parentheses are optional only if no required values are included.  

    @Exercise public class Cheetah { } // DOES NOT COMPILE. The hoursPerDay is a required value. While the annotation itself is optional,
                                       //                   the compiler still cares that they are used correctly.
    
When declaring an annotation, any element without a default value is considered required.

### Providing an Optional Element

For an element to be optional, rather than required, it must include a default value. For example:
  
    public @interface Exercise {
      int hoursPerDay();
      int startHour() default 6;
    }
  
Next, let's apply the updated annotation to our classes:
    
    @Exercise(hoursPerDay=3, startHour=5) public class Cheetah { } // COMPILES

    @Exercise(hoursPerDay=3) public class Cheetah { } // COMPILES. It will be instantiated with the default value of 6.

    @Exercise(hoursPerDay=3, startHour="5") public class Cheetah { } // DOES NOT COMPILE. It defines a value that is incompatible with the int type of startHour.
    
When we have more than one element value within an annotation, we separate them by a comma (,). Each element is written using the syntax elementName=elementValue. It's like a shorthand for a Map. The order of each element does not matter. 

### Defining a Default Element Value

The default value of an annotation element cannot be just any value. Similar to case statement values, the default value of an annotation must be a non-null constant expression. Example: 
    
    public @interface BadAnnotation {
      String name() default new String(""); // DOES NOT COMPILE. It does not defines a constant expression.
      String address() default ""; // COMPILES
      String title() default null; // DOES NOT COMPILE. Is null..
    }
    
  
### Selecting an Element Type

An annotation element cannot be declared with just any type, similar to a default element value. It must be a primitive type, a String, a Class (type Class), an enum, another annotation or an array of any of these types. For example:
    
    public class Bear { }

    public enum Size { SMALL, MEDIUM, LARGE }

    public @interface Panda {
      Integer height();   // DOES NOT COMPILE. While primitives types like int and long are supported, wrapper classes like Integer and Long are not.
      String[][] generalInfo();   // DOES NOT COMPILE. The type String[] is supported, as it is an array of String values, but String[][] is not.
      Bear friendlyBear();    // DOES NOT COMPILE. The type of this element is Bear (not Class). Even if Bear were changed to an interface, would still not compile.
      Size size() default Size.SMALL;   // COMPILES
      Exercise exercise() default @Exercise(hoursPerDay=2);   // COMPILES
    }
    

### Applying Element Modifiers

Like interface abstract methods, annotation elements are implicitly abstract and public, whether you declare them that way or not. Example:
    
    public @interface Fluffy {
      int cuteness();   // COMPILES
      public abstract int softness() default 11;   // COMPILES
      protected String material();   // DOES NOT COMPILE. 
      private String friendly();   // DOES NOT COMPILE. The access modifier on both material() and friendly() conflicts with the elements being implicitly public.
      final boolean isBunny();   // DOES NOT COMPILE. Like abstract methods, it cannot be marked final.
    }
    

### Adding a Constant variable

Annotations can include constant variables that can be accessed by other classes without actually creating the annotation. Example of an annotation constant:
    
    public @interface ElectricitySource { // This @interface compiles normally.
      public int voltage();
      int MIN_VOLTAGE = 2;
      public static final int MAX_VOLTAGE = 18;
    } 
    
Just like interface variables, annotation variables are implicitly public, static and final. These constant variables are not considered elements. For example, marker annotations can contain constants.

## Applying annotations (p.563-568)
Let's discuss other ways of applying annotations. Annotations can be applied to any Java declaration including the following:

- Classes, interfaces, enums and modules
- Variables (static, instance and local)
- Methods and constructors
- Method, constructor and lambda parameters
- Cast expressions
- Other annotations
  
The following example compiles, assuming the annotations referenced in it exist:
    
    @FunctionalInterface interface Speedster {
      void go(String name);
    }

    @LongEars
    @Soft @Cuddly public class Rabbit {
      @Deprecated public Rabbit(@NotNull Integer size) { }

      @Speed(velocity="fast") public void eat(@Edible String input) {
        @Food(vegetarian=true) String m = (@Tasty String) "carrots";

        Speedster s1 = new @Racer Speedster() {
          public void go(@FirstName @NotEmpty String name) {
            System.out.print("Start!" + name);
          }
        };

        Speedster s2 = (@Valid String n) -> System.out.print(n);
      }
    }
    
When applying an annotation to an expression, a cast operation including the Java type is required. On the line of declaration of the local variable 'String m', the expression was cast to String and the annotation @Tasty was applied to the type. Remember when mixing required and optional elements: `to use an annotation, all required values must be provided`. While an annotation may have many elements, values are required only for ones without default values.
    
    public @interface Swimmer {
      int armLength = 10; // CONSTANT, it cannot be included in an annotation (when applying her).
      String stroke(); // REQUIRED
      String name(); // REQUIRED
      String favoriteStroke() default "Backstroke"; // OPTIONAL
    }
    
### Creating a value() Element

You may have seen an annotation with a value, written without the elementName. The following syntax is valid under the right condition:

	`@Injured("Broken Tail") public class Monkey { }`

This is considered a shorthand or abbreviated annotation notation. An annotation must adhere to the following rules to be used without a name:

1. The annotation declaration must contain an element named value(), which may be optional or required.
2. The annotation declaration must not contain any other elements that are required.
3. The annotation usage must not provide values for any other elements.

For example:
    
    public @interface Injured {
      String veterinarian() default "unassigned";
      String value() default "foot"; // VALID
      int age() default 1;
    }

    public abstract class Elephant {
      @Injured("Legs") public void fallDown() { } // Annotation value goes to the element value()
      @Injured(value = "Legs") public abstract int trip();
      @Injured String injuries[];
    }
    
The usage in the first two annotation usages are equivalent, as the compiler will convert the shorthand form to the long form with the value() element name.

> **Good coding practice note:** Typically, the value() of an annotation should be related to the name of the annotation. In our previous example, @Injured was the annotation name and the value() referred to the item that was impacted. This is especially important since all shorthand elements use the same element name.

For the exam, make sure that if the shorthand notation is used, then there is an element named value(). Also, check if there are no other required elements. For example the following declarations cannot be used with a shorthand annotation:
    
    public @interface Sleep {
      int value();
      String hours(); // Even with a value() element, it cannot be used, since there are two required elements declared. 
    }

    public @interface Wake {
      String hours(); // Not named value()
    }
    
Likewise, the following annotation is not valid as it provides more than one value:

	`@Injured("Fur", age=2) public class Bear {} // DOES NOT COMPILE!`
  
### Passing an Array of Values

Annotations support a shorthand notation for providing an array that contains a single element. For example:
    
    public @interface Music {
      String[] genres();
    }
    
If we want to provide only one value to the array, we have a choice of two ways to write the annotation. Either of the following is correct:
    
    public class Giraffe {
      @Music(genres={"Rock and roll"}) String mostDisliked;
      @Music(genres="Classical") String favorite;
    }
    
The first usage is considered the regular form. The second annotation is the shorthand notation. Keep in mind that this is still providing a value for an array element; the compiler is just inserting the missing array braces for you. This notation can be used only if the array is composed of a single element. For example, only one of the following compiles:
    
    public class Reindeer {
      @Music(genres="Blues", "Jazz") String favorite; // DOES NOT COMPILE, more than one value
      @Music(genres=) String mostDisliked; // DOES NOT COMPILE, array without any values
      @Music(genres=null) String other; // DOES NOT COMPILE, array without any values
      @Music(genres={}) String alternative; // COMPILES, an array with no elements is still a valid array
    }
    
Remember that List or Collection are not in the list of supported element types for annotations. We can combine shorthand notations! For example, combine a shorthand notation value() and an array notation. For example, the following annotations are valid:
    
    // Rhythm single element is String[] value();
    public class Capybara {
      @Rhythm(value={"Swing"}) String favorite;
      @Rhythm(value="R&B") String secondFavorite;
      @Rhythm({"Classical"}) String mostDisliked;
      @Rhythm("Country") String lastDisliked;
    }
     
## Declaring Annotation-Specific Annotations (p.568-576)

Now we'll cover built-in annotations applied to other annotations. Yes, metadata about metadata. Since these annotations re built into Java, they primarily impact the compiler.

### Limiting Usage with @Target 
Many annotation declarations include @Target annotation, which limits the types the annotation can be applied to. More specifically, the @Target annotation takes an array of ElementType enuim values as its value() element. Values for the @Target annotations, ElementType value followed by what they apply to:

- TYPE: Classes, interfaces, enums, annotations.
- FIELD: Instance and static variables, enum values.
- METHOD: Method declarations.
- PARAMETER: Constructor, method and lambda parameters.
- CONSTRUCTOR: Constructor declarations.
- LOCAL_VARIABLE: Local variables.
- ANNOTATION_TYPE: Annotations.
- PACKAGE: Packages declared in package-info.java (out of scope for the exam, but is good to know).
- TYPE_PARAMETER: Parameterized types, generic declarations.
- TYPE_USE: Able to be applied anywhere there is a Java type declared or used.
- MODULE: Modules (out of scope for the exam, but is good to know).

> **Note on PACKAGE ElementType:** You can't add a package annotation to just any package declaration, only those defined in a special file, which must be named package-info.java. This file stores documentation metadata about a package. This isn't on the exam.

You might notice that some of the ElementType applications overlap. For example, you could declare an @Target with ANNOTATION_TYPE or TYPE. Either will work for annotations, although the second option opens the annotation usage to other types like classes and interfaces. Example of application:
    
    @Target({ElementType.METHOD, ElementType.CONSTRUCTOR})
    public @interface ZooAttraction { }

    @ZooAttraction class RollerCoaster { } // DOES NOT COMPILE, can't be applied to a class type.
    class Events {
      @ZooAttraction String rideTrain() { // COMPILES, method declaration is permitted.
        return (@ZooAttraction String) "Fun!"; // DOES NOT COMPILE, It's not permitted on a cast operation.
      }

      @ZooAttraction // COMPILES, contructor declaration is permitted.
      Events(@ZooAttraction String description) { // DOES NOT COMPILE ANYMORE, since constructor parameters are not permitted.
        super();
      }
      @ZooAttraction int numPassengers; // DOES NOT COMPILE, fields or variables are not permitted on this annotation.
    }
    
> **Note on importing these annotations elements:** Even though the java.lang package is imported automatically by the compile, the java.lang.annotation package is not.  Therefore, import statements are required for many of the examples in the remainder of this chapter.

### Understanding the TYPE_USE Value

The TYPE_USE is the most complex of those listed types. The TYPE_USE parameter can be used anywhere there is a Java type. By including it in @Target, it actually includes nearly all the values listed before, included classes, interfaces, constructors, parameters and more. There are a few exceptions, for example: 
- It can be used only on a method that returns a value. Methods that return void would still need the METHOD value defined.
- It also allows annotations in places where types are used, such as cast operations, object creation with new, inside type declarations, etc.

The following are valid TYPE_USE applications:
    
    // Technical.java
    import java.lang.annotation.ElementType;
    import java.lang.annotation.Target;
    
    @Target(ElementType.TYPE_USE)
    @interface Technical { }

    // NetworkRepair.java
    import java.util.function.predicate;

    public class NetworkRepair {

      class OutSrc extends @Technical NetworkRepair { }

      public void repair() {
        var repairSubclass = new @Technical NetworkRepair() { };

        var o = new @Technical NetworkRepair().new @Technical OutSrc();

        int remaining = (@Technical int)10.0;
      }
    }
    
For the exam, you don't need to know all of the places and what applyting it to these locations actually does. But you do need to recognize that they can be applied in this manner if TYPE_USE is one of the @Target options.

### Storing Annotations with @Retention

As you saw in Chapter 7, "Methods and Encapsulation", the compiler discard certain types of information when converting your source code into a .class file. With generics, this is known as type erasure. In a similar vein, annotations MAY be discarded by the compiler or at runtime. We say "may", because we can actually specify how they are handled using the @Retention annotation. This annotation takes a value() of the enum RetentionPolicy. The following are the RetentionPolicy values in increasing order of retention:

- SOURCE: Used only in the source file, discarded by the compiler.
- CLASS: Stored in the .class file but not available at runtime (default compiler behavior).
- RUNTIME: Stored in the .class file and available at runtime.
  
Examples of @Retention application:
    
    import java.lang.annotation.Retention;
    import java.lang.annotation.RetentionPolicy;

    @Retention(RetentionPolicy.CLASS) @interface Flier { }
    @Retention(RetentionPolicy.RUNTIME) @interface Swimmer { }
    
In this example, both annotation will retain the annotation info in their .class file, although only Swimmer will be available (via reflection) at runtime.

### Generating Javadoc with @Documented

Remember, Javadoc is a built-in standard within Java that generates documentation for a class or API. You can generate Javadoc files for any class you write. You can add additional metadata, including comments and annotations, that have no impact on your code but provide more detailed and user-friendly Javadoc files.

You should be familiar with the @Documented 'marker annotation'. If present, then the generated Javadoc will include annotation information defined on Java types. Because it is a marker annotation, it doesn't take any values; therefore, using it is pretty easy:
    
    // Hunter.java
    import java.lang.annotation.Documented;
    @Documented public @interface Hunter { }

    //Lion.java
    @Hunter public class Lion { }
    
The @Hunter annotation would be published with the Lion Javadoc information because it's marked with the @Documented annotation.

### Java vs. Javadoc Annotations

Javadoc has its own annotations that are used solely in generating data within a Javadoc file.
    
    public class ZooLightShow { 

      /**
       *  Performs a light show at the zoo.
       *
       *  @param  distance  length the light needs to travel.
       *  @return   the result of the light show operation. 
       *  @author   Grace Hopper
       *  @since    1.5
       *  @deprecated Use EnchancedZooLightShow.lights() instead.
       */
       @Deprecated(since="1.5") public static String perform(int distance) {
         return "Beginning light show!";
       } 
    }
    
Be careful not to confuse Javadoc annotations with the Java annotations. Take a look at the @deprecated and @Deprecated annotations in this example. The first, @deprecated, is a Javadoc annotation used inside a comment, while @Deprecated is a Java annotation applied to a class.

> **Note:** Traditionally, Javadoc annotations are all lowercase, while Java annotations start with an uppercase letter.

### Inheriting Annotations with @Inherited

Another 'marker annotation' you should know for the exam is @Inherited. When this annotation is applied to a class, subclasses will inherit the annotation information found in the parent class. For example:
    
    // Vertebrate.java
    import java.lang.annotation.Inherited;
    @Inherited public @interface Vertebrate { }

    // Mammal.java
    @Vertebrate public class Mammal { }

    // Dolphin.java
    public class Dolphin extends Mammal { }
    
In this example, the @Vertebrate annotation will be applied to both Mammal and Dolphin objects (because of the @Inhertied annotation). 

### Supporting Duplicates with @Repeatable

The @Repeatable annotations is used when you want to specify an annotation more than once on a Java type. Is arguably the most complicated to use, as it actually requires creating two annotations to work. Generally, you use repeatable annotations when you want to apply the same annotation with different values in the same type. We need to follow this rules when creating repeatable annotations:
1. Without the @Repeatable annotation, an annotation can be applied only once.
2. The repeatable annotation must be declared with @Repeatable and contain a value that refers to the containing type annotation.
3. The containing type annotation must include an element named value(), which is a primitive array of the repeatable annotation type.
   
A 'containing annotation type' is a separate annotation that defines a value() array element. The type of this array is the particular annotation you want to repeat. By convention, the name of the annotation is often the plural form of the repeatable annotation. The following is the right/best way to declare and apply an annotation that is annotated with @Repeatable:
    
    // Risks.java - Containing annotation type
    public @interface Risks { 
      Risk[] value();
    }

    // Risk.java
    @Repeatable(Risks.class)
    public @interface Risk {
      String danger();
      int level() default 1;
    }

    // Zoo.java
    public class Zoo {
      public static class Monkey { }

      @Risk(danger="Silly")
      @Risk(danger="Agressive", level=5)
      @Risk(danger="Violent", level=10)
      private Monkey monkey;
    }
    
Notice that we never really use the @Risks in our Zoo class, but given the declaration of the Risk and Risks annotations, the compiler takes care of applying the annotations for us.

### Repeatable Annotations vs. an Array of Annotations

Repeatable annotations were added in Java 8. Prior to this, you would have to use the @Risks containing annotation type directly:
    
    @Risks({
      @Risk(danger="Silly")
      @Risk(danger="Agressive", level=5)
      @Risk(danger="Violent", level=10)
    })
    private Monkey monkey;
    
With this implementation, the @Repeatable is not required in the Risk annotation declaration. But using the @Repeatable annotation is the preferred approach now, as it is easier than working with multiple nested statements.

### Reviewing Annotation-Specific Annotations

Annotations that can be applied to other annotation that might appear on the exam (they are common in real world scenarios too):
   
  |Annotation  |  Marker Annotation  |  Type of value()      |    Default compiler behavior (if annnotation not present)                            |
  |:---------  | :-----------------  | :---------------      |   :---------------------------------------------------                               |
  |@Target     |  No                 | Array of ElementType  |   Annotation able to be applied to all locations except TYPE_USE and TYPE_PARAMETER  |
  |@Retention  |  No                 | RetentionPolicy       |   RetentionPolicy.CLASS                                                              |
  |@Documented |  Yes                |  -                    |   Annotation are not included in the generated Javadoc                               |
  |@Inherited  |  Yes                |  -                    |   Annotations in supertypes are not inherited                                        |
  |@Repeatable |  No                 | Annotation            |   Annotation cannot be repeated                                                      |

Prior to this section, we created numerous annotations and we never used any of the annotation listed in the table. So, what did the compiler do? Like implicit modifiers and default no-arg constructors, the compiler auto-inserted information based on the lack of data.

The default behavior for most of the annotations in the table is often intuitive. For example, without the @Documented or @Inherited annotation, these features are not supported.

If you try to use an annotation more than once without the @Repeatable annotation, the compiler will report an error.

@Target is a bit of a special case, here's why:
- When @Target is not present, an annotation can be used in any place except TYPE_USE or TYPE_PARAMETER scenarios (cast operations, object creation, generic 
declarations, etc). So to use an annotation in a type use or type parameter, like lambdas or generic declaration, you must explicitly set the @Target to include these values. If an annotation is declared without the @Target annotation that includes these values, then these locations are prohibited.
- One possible explanation for this behavior is backward compatibility. When these values were added in Java 8, it was decided that they would have to be explicitly declared to be used in these locations.
- With that said, when the authors of Java added the MODULE value in Java 9, they didn't make the this same decision. So if @Target is absent, the annotation is permitted in amodule declaration by default.