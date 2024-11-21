# Minimize the accessibility of classes and members

The single most important factor that distinguishes a well-designed component from a poorly designed one is the degree to which the component hides its internal data and other implementation details from other components. A well-designed component hides all its implementation details, cleanly separating its API from its implementation. Components then communicate only through their APIs and are `oblivious(不关心)` to each others’ inner workings. This concept, known as *information hiding* or *encapsulation*, is a fundamental `tenet(原则、信条)` of software design [Parnas72].

Information hiding is important for many reasons, `most of which stem(大部分都源于)` from the fact that it *decouples* the components that `comprise(组成)` a system, allowing them to be developed, tested, optimized, used, understood, and modified in isolation. This speeds up system development because components can be developed in parallel. `It eases (减轻)`the `burden(负担)` of `maintenance(维护)` because components can be understood more quickly and debugged or replaced with little `fear(担心、害怕)` of `harming(损害、伤害)` other components. While information hiding does not, in and of itself, cause good performance, it enables effective performance tuning: once a system is complete and `profiling(分析)` has determined which components are causing performance problems (Item 67), those components can be optimized without affecting the `correctness(正确性)` of others. Information hiding increases software reuse because components that aren’t tightly coupled often `prove(被证明)` useful in other contexts besides the ones for which they were developed. Finally, information hiding decreases the risk in building large systems because individual components may prove successful even if the system does not. Java has many facilities to aid in information hiding. The *access control* mechanism specifies the *accessibility* of classes, interfaces, and members. The accessibility of an entity is determined by the location of its declaration and `by which(由)`, if any, of the access modifiers (private, protected, and public) is present on the declaration. Proper use of these modifiers is essential to information hiding.

> Why need information hiding?
>
> * It decouples the components that comprise a system.
> * It eases the burden of maintnance.
> * It enables effective performance tuning.
> * It increases software reuse.
> * It decreases the risk in building large systems.

The rule of thumb is simple: **make each class or member as inaccessible as possible.** In other words, use the lowest possible access level consistent with the proper functioning of the software that you are writing.

For top-level (non-nested) classes and interfaces, there are only two possible access levels: *package-private* and *public*. If you declare a top-level class or interface with the public modifier, it will be public; otherwise, it will be package- private. If a top-level class or interface can be made package-private, it should be. By making it package-private, you make it part of the implementation rather than the exported API, and you can modify it, replace it, or `eliminate(消除)` it in a subsequent release without fear of harming existing clients. If you make it public, you are `obligated(有义务)` to support it forever to maintain compatibility.

If a package-private top-level class or interface is used by only one class, consider making the top-level class a private static nested class of the sole class that uses it (Item 24). This reduces its accessibility from all the classes in its package to the one class that uses it. But it is far more important to reduce the accessibility of a `gratuitously(不必要的)` public class than of a package-private top-level class: the public class is part of the package’s API, while the package-private top-level class is already part of its implementation.

For members (fields, methods, nested classes, and nested interfaces), there are four possible access levels, listed here in order of increasing accessibility:

- **private**—The member is accessible only from the top-level class where it is declared.
- **package-private**—The member is accessible from any class in the package where it is declared. Technically known as *default* access, this is the access level you get if no access modifier is specified (except for interface members, which are public by default).
- **protected**—The member is accessible from subclasses of the class where it is declared (subject to a few restrictions [JLS, 6.6.2]) and from any class in the package where it is declared.
- **public**—The member is accessible from anywhere.

After carefully designing your class’s public API, your `reflex(本能反应)` should be to make all other members private. Only if another class in the same package really needs to access a member should you remove the private modifier, making the member package-private. If you find yourself doing this often, you should re-examine the design of your system to see if another `decomposition(分解)` might `yield(产生)` classes that are better decoupled from one another. That said, both private and package-private members are part of a class’s implementation and do not normally `impact(冲击、挤入、产生影响)` its exported API. These fields can, however, “leak” into the exported API if the class implements Serializable (Items 86 and 87).

For members of public classes, a huge increase in accessibility occurs when the access level goes from package-private to protected. A protected member is part of the class’s exported API and must be supported forever. Also, a protected member of an exported class represents a public `commitment(承诺、保证)` to an implementation detail (Item 19). The need for protected members should be `relatively(相对) rare(稀少)`.

There is a key rule that `restricts(限制) `your ability to reduce the accessibility of methods. If a method overrides a superclass method, it cannot have a more `restrictive(限定性) `access level in the subclass than in the superclass [JLS, 8.4.8.3]. This is necessary to ensure that an instance of the subclass is usable anywhere that an instance of the superclass is usable (the *Liskov substitution principle*, see Item 15). If you violate this rule, the compiler will generate an error message when you try to compile the subclass. A special case of this rule is that if a class implements an interface, all of the class methods that are in the interface must be declared public in the class.

To `facilitate(使更容易)` testing your code, you may be tempted to make a class, interface, or member more accessible than otherwise necessary. This is fine `up to a point(在一定程度上)`. It is acceptable to make a private member of a public class package-private in order to test it, but it is not acceptable to raise the accessibility any higher. In other words, it is not acceptable to make a class, interface, or member a part of a package’s exported API to facilitate testing. Luckily, it isn’t necessary either because tests can be made to run as part of the package being tested, thus gaining access to its package-private elements.

**Instance fields of public classes should rarely be public** (Item 16). If an instance field is nonfinal or is a reference to a mutable object, then by making it public, you give up the ability to limit the values that can be stored in the field. This means you give up the ability to `enforce(强迫)``invariants(不变性)`involving the field. Also, you give up the ability to take any action when the field is modified, so **classes with public mutable fields are not generally thread-safe.** Even if a field is final and refers to an immutable object, by making it public you give up the flexibility to switch to a new internal data representation in which the field does not exist.

The same advice applies to static fields, with one `exception(例外)`. You can expose constants via public static final fields, assuming the constants form an `integral(必须的) `part of the abstraction provided by the class. By convention, such fields have names consisting of `capital(大写的)` letters, with words separated by underscores (Item 68). It is critical that these fields contain either primitive values or references to immutable objects (Item 17). a field containing a reference to a mutable object has all the disadvantages of a nonfinal field. While the reference cannot be modified, the referenced object can be modified—with disastrous results.

Note that a nonzero-length array is always mutable, so **it is wrong for a class to have a public static final array field, or an accessor that returns such a field.** If a class has such a field or accessor, clients will be able to modify the con- tents of the array. This is a frequent source of security holes:

```java
   // Potential security hole!
   public static final Thing[] VALUES =  { ... };
```

Beware of the fact that some IDEs generate accessors that return references to private array fields, resulting in exactly this problem. There are two ways to fix the problem. You can make the public array private and add a public immutable list:

```java
   private static final Thing[] PRIVATE_VALUES = { ... };
   public static final List<Thing> VALUES =
       Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

Alternatively, you can make the array private and add a public method that returns a copy of a private array:

```java
   private static final Thing[] PRIVATE_VALUES = { ... };
   public static final Thing[] values() {
       return PRIVATE_VALUES.clone();
   }
```

To choose between these alternatives, think about what the client is likely to do with the result. Which return type will be more convenient? Which will give better performance?

As of Java 9, there are two additional, implicit access levels introduced as part of the *module system*. A module is a grouping of packages, like a package is a grouping of classes. A module may explicitly export some of its packages via *export declarations* in its *module declaration* (which is by convention contained in a source file named module-info.java). Public and protected members of unexported packages in a module are inaccessible outside the module; within themodule, accessibility is unaffected by export declarations. Using the module system allows you to share classes among packages within a module without making them visible to the entire world. Public and protected members of public classes in unexported packages give rise to the two implicit access levels, which are intramodular analogues of the normal public and protected levels. The need for this kind of sharing is relatively rare and can often be eliminated by rearranging the classes within your packages.

Unlike the four main access levels, the two module-based levels are largely advisory. If you place a module’s JAR file on your application’s class path instead of its module path, the packages in the module revert to their non-modular behav- ior: all of the public and protected members of the packages’ public classes have their normal accessibility, regardless of whether the packages are exported by the module [Reinhold, 1.2]. The one place where the newly introduced access levels are strictly enforced is the JDK itself: the unexported packages in the Java libraries are truly inaccessible outside of their modules.

Not only is the access protection afforded by modules of limited utility to the typical Java programmer, and largely advisory in nature; in order to take advantage of it, you must group your packages into modules, make all of their dependencies explicit in module declarations, rearrange your source tree, and take special actions to accommodate any access to non-modularized packages from within your modules [Reinhold, 3]. It is too early to say whether modules will achieve widespread use outside of the JDK itself. In the meantime, it seems best to avoid them unless you have a compelling need.

To summarize, you should reduce accessibility of program elements as much as possible (within reason). After carefully designing a minimal public API, you should prevent any stray classes, interfaces, or members from becoming part of the API. With the exception of public static final fields, which serve as constants, public classes should have no public fields. Ensure that objects referenced by public static final fields are immutable.



# In public classes,use accessor methods,not public fields

Occasionally, you may be tempted to write degenerate classes that serve no pur- pose other than to group instance fields:

```
   // Degenerate classes like this should not be public!
   class Point {
       public double x;
       public double y;
   }
```

Because the data fields of such classes are accessed directly, these classes do not offer the benefits of *encapsulation* (Item 15). You can’t change the representa- tion without changing the API, you can’t enforce invariants, and you can’t take auxiliary action when a field is accessed. Hard-line object-oriented programmers feel that such classes are anathema and should always be replaced by classes with private fields and public *accessor methods* (getters) and, for mutable classes, *mutators* (setters):

```
   // Encapsulation of data by accessor methods and mutators
   class Point {
       private double x;
       private double y;
       public Point(double x, double y) {
           this.x = x;
```

this.y = y; }

```
       public double getX() { return x; }
       public double getY() { return y; }
       public void setX(double x) { this.x = x; }
       public void setY(double y) { this.y = y; }
```

}

Certainly, the hard-liners are correct when it comes to public classes: **if a class is accessible outside its package, provide accessor methods** to preserve the flexibility to change the class’s internal representation. If a public class exposes its data fields, all hope of changing its representation is lost because client code can be distributed far and wide.

However, **if a class is package-private or is a private nested class, there is nothing inherently wrong with exposing its data fields**—assuming they do an adequate job of describing the abstraction provided by the class. This approach generates less visual clutter than the accessor-method approach, both in the class definition and in the client code that uses it. While the client code is tied to the class’s internal representation, this code is confined to the package containing the class. If a change in representation becomes desirable, you can make the change without touching any code outside the package. In the case of a private nested class, the scope of the change is further restricted to the enclosing class.

Several classes in the Java platform libraries violate the advice that public classes should not expose fields directly. Prominent examples include the Point and Dimension classes in the java.awt package. Rather than examples to be emulated, these classes should be regarded as cautionary tales. As described in Item 67, the decision to expose the internals of the Dimension class resulted in a serious performance problem that is still with us today.

While it’s never a good idea for a public class to expose fields directly, it is less harmful if the fields are immutable. You can’t change the representation of such a class without changing its API, and you can’t take auxiliary actions when a field is read, but you can enforce invariants. For example, this class guarantees that each instance represents a valid time:

```
   // Public class with exposed immutable fields - questionable
   public final class Time {
       private static final int HOURS_PER_DAY    = 24;
       private static final int MINUTES_PER_HOUR = 60;
       public final int hour;
       public final int minute;
       public Time(int hour, int minute) {
           if (hour < 0 || hour >= HOURS_PER_DAY)
              throw new IllegalArgumentException("Hour: " + hour);
           if (minute < 0 || minute >= MINUTES_PER_HOUR)
```

throw new IllegalArgumentException("Min: " + minute); this.hour = hour;
 this.minute = minute;

}

```
       ... // Remainder omitted
   }
```

In summary, public classes should never expose mutable fields. It is less harmful, though still questionable, for public classes to expose immutable fields. It is, however, sometimes desirable for package-private or private nested classes to expose fields, whether mutable or immutable.



# Favor static member classes over nonstatic

A *nested class* is a class defined within another class. **A nested class should exist only to serve its enclosing class`（这个很重要，嵌套内部类的存在应该仅仅是为了服务于他的外部类）`**. If a nested class would be useful in some other context, then it should be a top-level class. 

There are four kinds of nested classes: 

* static member classes
*  nonstatic member classes
* anonymous classes 
*  local classes. 

All but the first kind are known as inner classes. This item tells you when to use which kind of nested class and why.

A static member class is the simplest kind of nested class. It is best thought of as an ordinary class that happens to be declared inside another class and has access to all of the enclosing class’s members, even those declared private. A static member class is a static member of its enclosing class and obeys the same accessibility rules as other static members. If it is declared private, it is accessible only within the enclosing class, `and so forth(等等)`.

One common use of a static member class is **as a public helper class**, useful only in `conjunction with(结合)` its outer class. For example, consider an enum describing the operations supported by a calculator. The Operation enum should be a public static member class of the Calculator class. Clients of Calculator could then refer to operations using names like Calculator.Operation.PLUS and Calculator.Operation.MINUS.

`Syntactically（在语法上）`, the only difference between static and nonstatic member classes is that static member classes have the modifier static in their declarations. Despite the syntactic similarity, these two kinds of nested classes are very different. Each instance of a nonstatic member class is implicitly associated with an *enclosing instance* of its containing class. Within instance methods of a nonstatic member class, you can invoke methods on the enclosing instance or obtain a reference to the enclosing instance using the qualified this construct. If an instance of a nested class can exist in isolation from an instance of its enclosing class, then the nested class *must* be a static member class: it is impossible to create an instance of a nonstatic member class without an enclosing instance.

The association between a nonstatic member class instance and its enclosing instance is established when the member class instance is created and cannot be modified `thereafter(之后)`. Normally, the association is established automatically by invoking a nonstatic member class constructor from within an instance method of the enclosing class. It is possible, though rare, to establish the association manually using the expression enclosingInstance.new MemberClass(args). As you would expect, the association takes up space in the nonstatic member class instance and adds time to its construction.

One common use of a nonstatic member class is to define an *Adapter* [Gamma95] that allows an instance of the outer class to be viewed as an instance of some unrelated class. For example, implementations of the Map interface typi- cally use nonstatic member classes to implement their *collection views*, which are returned by Map’s keySet, entrySet, and values methods. Similarly, implemen- tations of the collection interfaces, such as Set and List, typically use nonstatic member classes to implement their iterators:

```java
   // Typical use of a nonstatic member class
   public class MySet<E> extends AbstractSet<E> {
       ... // Bulk of the class omitted
       @Override public Iterator<E> iterator() {
           return new MyIterator();
           }

private class MyIterator implements Iterator<E>** { ...

} }
```



**If you declare a member class that does not require access to an enclosing instance,** **always** **put the** **static** **modifier in its declaration,** making it a static rather than a nonstatic member class. If you omit this modifier, each instance will have a hidden extraneous reference to its enclosing instance. As previously men- tioned, storing this reference takes time and space. More seriously, it can result in the enclosing instance being retained when it would otherwise be eligible for gar- bage collection (Item 7). The resulting memory leak can be catastrophic. It is often difficult to detect because the reference is invisible.

A common use of private static member classes is to represent components of the object represented by their enclosing class. For example, consider a Map instance, which associates keys with values. Many Map implementations have an internal Entry object for each key-value pair in the map. While each entry is asso- ciated with a map, the methods on an entry (getKey, getValue, and setValue) do not need access to the map. Therefore, it would be wasteful to use a nonstatic member class to represent entries: a private static member class is best. If you accidentally omit the static modifier in the entry declaration, the map will still work, but each entry will contain a superfluous reference to the map, which wastes space and time.

It is doubly important to choose correctly between a static and a nonstatic member class if the class in question is a public or protected member of an exported class. In this case, the member class is an exported API element and can- not be changed from a nonstatic to a static member class in a subsequent release without violating backward compatibility.

As you would expect, an anonymous class has no name. It is not a member of its enclosing class. Rather than being declared along with other members, it is simultaneously declared and instantiated at the point of use. Anonymous classes are permitted at any point in the code where an expression is legal. Anonymous classes have enclosing instances if and only if they occur in a nonstatic context. But even if they occur in a static context, they cannot have any static members other than *constant variables*, which are final primitive or string fields initialized to constant expressions [JLS, 4.12.4].

There are many limitations on the `applicability(适应性)` of anonymous classes. You can’t instantiate them except at the point they’re declared. You can’t perform instanceof tests or do anything else that requires you to name the class. You can’t declare an anonymous class to implement multiple interfaces or to extend a class and implement an interface at the same time. Clients of an anonymous class can’t invoke any members except those it inherits from its supertype. Because anonymous classes occur in the midst of expressions, they must be kept short— about ten lines or fewer—or readability will suffer.

Before lambdas were added to Java (Chapter 6), anonymous classes were the preferred means of creating small *function objects* and *process objects* on the fly, but lambdas are now preferred (Item 42). Another common use of anonymous classes is in the implementation of static factory methods (see intArrayAsList in Item 20).

Local classes are the least frequently used of the four kinds of nested classes. A local class can be declared practically anywhere a local variable can be declared and obeys the same scoping rules. Local classes have attributes in common with each of the other kinds of nested classes. Like member classes, they have names and can be used repeatedly. Like anonymous classes, they have enclosing instances only if they are defined in a nonstatic context, and they cannot contain static members. And like anonymous classes, they should be kept short so as not to harm readability.

To recap, there are four different kinds of nested classes, and each has its place. If a nested class needs to be visible outside of a single method or is too long to fit comfortably inside a method, use a member class. If each instance of a mem- ber class needs a reference to its enclosing instance, make it nonstatic; otherwise, make it static. Assuming the class belongs inside a method, if you need to create instances from only one location and there is a preexisting type that characterizes the class, make it an anonymous class; otherwise, make it a local class.





# Favor composition over inheritance

Inheritance is a powerful way to achieve code reuse, but it is not always the best tool for the job. Used inappropriately, it leads to `fragile(脆弱的)` software. It is safe to use inheritance within a package, where the subclass and the superclass implementations are under the control of the same programmers. It is also safe to use inheritance when extending classes specifically designed and documented for extension. Inheriting from ordinary concrete classes across package boundaries, however, is dangerous. As a reminder, this book uses the word “inheritance” to mean *implementation inheritance* (when one class extends another). The problems discussed in this item do not apply to *interface inheritance* (when a class implements an interface or when one interface extends another).

**Unlike method invocation, inheritance `violates(破坏了)` encapsulation**. In other words, a subclass depends on the implementation details of its superclass for its `proper(特有的、专有的)` function. The superclass’s implementation may change from release to release, and if it does, the subclass may break, even though its code has not been touched. As a `consequence(结果)`, a subclass must `evolve in tandem(同步发展)` with its superclass, unless the superclass’s authors have designed and documented it specifically for the purpose of being extended.

To make this concrete, let’s suppose we have a program that uses a HashSet. To tune the performance of our program, we need to query the HashSet as to how many elements have been added since it was created (not to be confused with its current size, which goes down when an element is removed). To provide this functionality, we write a HashSet `variant(变量)` that keeps count of the number of attempted element insertions and exports an accessor for this count. The HashSet class contains two methods capable of adding elements, add and addAll, so we override both of these methods:

```java
   // Broken - Inappropriate use of inheritance!
   public class InstrumentedHashSet<E> extends HashSet<E> {
     // The number of attempted element insertions
     private int addCount = 0;
     public InstrumentedHashSet() {
     }
     public InstrumentedHashSet(int initCap, float loadFactor) {
       super(initCap, loadFactor);
     }
     @Override 
     public boolean add(E e) {
        addCount++;
        return super.add(e);
		 }
     
      @Override public boolean addAll(Collection<? extends E> c) { 
        addCount += c.size();
        return super.addAll(c);
      }
     
      public int getAddCount() {
          return addCount;
      }
}
```

This class looks reasonable, but it doesn’t work. Suppose we create an instance and add three elements using the addAll method. Incidentally, note that we create a list using the static factory method List.of, which was added in Java 9; if you’re using an earlier release, use Arrays.asList instead:

```java
   InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
   s.addAll(List.of("Snap", "Crackle", "Pop"));
```

We would expect the getAddCount method to return three at this point, but it returns six. What went wrong? Internally, HashSet’s addAll method is implemented on top of its add method, although HashSet, quite reasonably, does not document this implementation detail. The addAll method in InstrumentedHashSet added three to addCount and then invoked HashSet’s addAll implementation using super.addAll. This `in turn(依次)` invoked the add method, as overridden in InstrumentedHashSet, once for each element. Each of these three invocations added one more to addCount, for a total increase of six: each element added with the addAll method is double counted.

We could “fix” the subclass by eliminating its override of the addAll method. While the resulting class would work, it would depend for its proper function on the fact that HashSet’s addAll method is implemented on top of its add method. This “self-use” is an implementation detail, not guaranteed to hold in all imple- mentations of the Java platform and subject to change from release to release. Therefore, the resulting InstrumentedHashSet class would be fragile.

It would be slightly better to override the addAll method to iterate over the specified collection, calling the add method once for each element. This would guarantee the correct result whether or not HashSet’s addAll method were implemented atop its add method because HashSet’s addAll implementation would no longer be invoked. This technique, however, does not solve all our problems. It amounts to reimplementing superclass methods that may or may not result in self-use, which is difficult, time-consuming, error-prone, and may reduce performance. Additionally, it isn’t always possible because some methods cannot be implemented without access to private fields inaccessible to the subclass.

A related cause of fragility in subclasses is that their superclass can acquire new methods in subsequent releases. Suppose a program depends for its security on the fact that all elements inserted into some collection satisfy some predicate. This can be guaranteed by subclassing the collection and overriding each method capable of adding an element to ensure that the predicate is satisfied before adding the element. This works fine until a new method capable of inserting an element is added to the superclass in a subsequent release. Once this happens, it becomes possible to add an “illegal” element merely by invoking the new method, which is not overridden in the subclass. This is not a purely theoretical problem. Several security holes of this nature had to be fixed when Hashtable and Vector were ret- rofitted to participate in the Collections Framework.

Both of these problems stem from overriding methods. You might think that it is safe to extend a class if you merely add new methods and refrain from overriding existing methods. While this sort of extension is much safer, it is not without risk. If the superclass acquires a new method in a subsequent release and you have the bad luck to have given the subclass a method with the same signature and a different return type, your subclass will no longer compile [JLS, 8.4.8.3]. If you’ve given the subclass a method with the same signature and return type as the new superclass method, then you’re now overriding it, so you’re subject to the problems described earlier. Furthermore, it is doubtful that your method will fulfill the contract of the new superclass method, because that contract had not yet been written when you wrote the subclass method.

Luckily, there is a way to avoid all of the problems described above. Instead of extending an existing class, give your new class a private field that references an instance of the existing class. This design is called *composition* because the exist- ing class becomes a component of the new one. Each instance method in the new class invokes the corresponding method on the contained instance of the existing class and returns the results. This is known as *forwarding*, and the methods in the new class are known as *forwarding methods*. The resulting class will be rock solid, with no dependencies on the implementation details of the existing class. Even adding new methods to the existing class will have no impact on the new class. To make this concrete, here’s a replacement for InstrumentedHashSet that uses the composition-and-forwarding approach. Note that the implementation is broken into two pieces, the class itself and a reusable *forwarding class,* which contains all of the forwarding methods and nothing else:

```java
// Wrapper class - uses composition in place of inheritance
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    public int getAddCount() {
        return addCount;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
}

// Reusable forwarding class
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;

    public ForwardingSet(Set<E> s) {
        this.s = s;
    }

    public void clear() {
        s.clear();
    }

    public boolean contains(Object o) {
        return s.contains(o);
    }

    public boolean isEmpty()

    public int size()

    public Iterator<E> iterator()

    public boolean add(E e)

    public boolean remove(Object o)

    public boolean containsAll(Collection<?> c) {
        return s.containsAll(c);
    }

    public boolean addAll(Collection<? extends E> c) {
        return s.addAll(c);
    }

    public boolean removeAll(Collection<?> c) {
        return s.removeAll(c);
    }

    public boolean retainAll(Collection<?> c) {
        return s.retainAll(c);
    }

    public Object[] toArray() {
        return s.toArray();
    }

    public <T> T[] toArray(T[] a) {
        return s.toArray(a);
    }

    @Override
    public boolean equals(Object o) {
        return s.equals(o);
    }

    @Override
    public int hashCode() {
        return s.hashCode();
    }

    @Override
    public String toString() {
        return s.toString();
    }
}
......
```



The design of the InstrumentedSet class is enabled by the existence of the Set interface, which captures the functionality of the HashSet class. Besides being robust, this design is extremely flexible. The InstrumentedSet class imple- ments the Set interface and has a single constructor whose argument is also of type Set. In essence, the class transforms one Set into another, adding the instru- mentation functionality. Unlike the inheritance-based approach, which works only for a single concrete class and requires a separate constructor for each supported constructor in the superclass, the wrapper class can be used to instrument any Set implementation and will work in conjunction with any preexisting constructor:

```java
   Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
   Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

The InstrumentedSet class can even be used to temporarily instrument a set instance that has already been used without instrumentation:

```java
   static void walk(Set<Dog> dogs) {
       InstrumentedSet<Dog> iDogs = new InstrumentedSet<>(dogs);
       ... // Within this method use iDogs instead of dogs
   }
```

The InstrumentedSet class is known as a *wrapper* class because each InstrumentedSet instance contains (“wraps”) another Set instance. This is also known as the *Decorator* pattern [Gamma95] because the InstrumentedSet class “decorates” a set by adding instrumentation. Sometimes the combination of com- position and forwarding is loosely referred to as *delegation.* Technically it’s not delegation unless the wrapper object passes itself to the wrapped object [Lieber- man86; Gamma95].

The disadvantages of wrapper classes are few. One caveat is that wrapper classes are not suited for use in *callback frameworks*, wherein objects pass self- references to other objects for subsequent invocations (“callbacks”). Because a wrapped object doesn’t know of its wrapper, it passes a reference to itself (this) and callbacks elude the wrapper. This is known as the *SELF problem* [Lieberman86]. Some people worry about the performance impact of forwarding method invocations or the memory footprint impact of wrapper objects. Neither turn out to have much impact in practice. It’s tedious to write forwarding methods, but you have to write the reusable forwarding class for each interface only once, and forwarding classes may be provided for you. For example, Guava provides forwarding classes for all of the collection interfaces [Guava].

Inheritance is appropriate only in circumstances where the subclass really is a *subtype* of the superclass. In other words, a class *B* should extend a class *A* only if an “is-a” relationship exists between the two classes. If you are tempted to have a class *B* extend a class *A*, ask yourself the question: Is every *B* really an *A*? If you cannot truthfully answer yes to this question, *B* should not extend *A*. If the answer is no, it is often the case that *B* should contain a private instance of *A* and expose a different API: *A* is not an essential part of *B*, merely a detail of its implementation.

There are a number of obvious violations of this principle in the Java platform libraries. For example, a stack is not a vector, so Stack should not extend Vector. Similarly, a property list is not a hash table, so Properties should not extend Hashtable. In both cases, composition would have been preferable.

If you use inheritance where composition is appropriate, you needlessly expose implementation details. The resulting API ties you to the original imple- mentation, forever limiting the performance of your class. More seriously, by exposing the internals you let clients access them directly. At the very least, it can lead to confusing semantics. For example, if p refers to a Properties instance, then p.getProperty(key) may yield different results from p.get(key): the for- mer method takes defaults into account, while the latter method, which is inher- ited from Hashtable, does not. Most seriously, the client may be able to corrupt invariants of the subclass by modifying the superclass directly. In the case of Properties, the designers intended that only strings be allowed as keys and val- ues, but direct access to the underlying Hashtable allows this invariant to be vio- lated. Once violated, it is no longer possible to use other parts of the Properties API (load and store). By the time this problem was discovered, it was too late to correct it because clients depended on the use of non-string keys and values.

There is one last set of questions you should ask yourself before deciding to use inheritance in place of composition. Does the class that you contemplate extending have any flaws in its API? If so, are you comfortable propagating those flaws into your class’s API? Inheritance propagates any flaws in the superclass’s API, while composition lets you design a new API that hides these flaws.

To summarize, inheritance is powerful, but it is problematic because it violates encapsulation. It is appropriate only when a genuine subtype relationship exists between the subclass and the superclass. Even then, inheritance may lead to fragility if the subclass is in a different package from the superclass and the superclass is not designed for inheritance. To avoid this fragility, use composition and forwarding instead of inheritance, especially if an appropriate interface to implement a wrapper class exists. Not only are wrapper classes more robust than subclasses, they are also more powerful.