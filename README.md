# Java Object Layout (JOL) plugin for IntelliJ Idea

[JOL](https://github.com/openjdk/jol/) (Java Object Layout) is the tool to analyze object layout schemes in JVMs.
For example, in HotSpot VM on 64x processor an empty string takes 40 bytes i.e. 24 bytes for String object itself + 16 bytes for an internal empty char array.

The plugin is a GUI for JOL and allows you to make an estimate how much memory the object takes.

Set a cursor into a class name and then press `Code / Show Object Layout` and you'll see a right panel with layout info.

![screenshot.png](screenshot.png)

Thus, you can perform simplest but most efficient performance improvements.
Just check your DTOs if they fit into 64 bytes of processor's cache line.

Only HotSpot VM is supported by JOL itself.
The plugin supports only basic estimate of class layout in different VM modes i.e. the same as `jol-cli estimates` command.
For more precise estimate use JOL library and estimate in run time on the real objects with `GraphLayout`:

```java
import org.openjdk.jol.info.GraphLayout;
import java.util.HashMap;

public class JolTest {

    public static void main(String[] args) {
        HashMap<Object, Object> hashMap = new HashMap<>();
        hashMap.put("key", "value");
        System.out.println(GraphLayout.parseInstance(hashMap).toFootprint());
    }
}
```

Output will be like:

    java.util.HashMap@7a79be86d footprint:
         COUNT       AVG       SUM   DESCRIPTION
             2        24        48   [B
             1        80        80   [Ljava.util.HashMap$Node;
             2        24        48   java.lang.String
             1        48        48   java.util.HashMap
             1        32        32   java.util.HashMap$Node
             7                 256   (total)

So you can see the full size including inner objects.

**NOTE:** Your app most likely will use the HotSpot with `64-bit VM, compressed references` mode. 

## Installation

- Using IDE built-in plugin system:

  <kbd>Settings/Preferences</kbd> > <kbd>Plugins</kbd> > <kbd>Marketplace</kbd> > <kbd>Search for "JOL"</kbd> >
  <kbd>Install Plugin</kbd>

- Manually:

  Download the [latest release](https://github.com/stokito/IdeaJol/releases/latest) and install it manually using
  <kbd>Settings/Preferences</kbd> > <kbd>Plugins</kbd> > <kbd>⚙️</kbd> > <kbd>Install plugin from disk...</kbd>


### Inspection
The plugin provides an inspection to see most big classes. It's enabled by default.
You can find the inspection by path `Java | Memory | JOL: Class has too big memory footprint` to configure or disable it. 

Please leave a feedback for the [plugin in marketplace](https://plugins.jetbrains.com/plugin/10953-java-object-layout).


## Tutorials
* [Java Objects Inside Out](https://shipilev.net/jvm/objects-inside-out/) from the JOL author.
* Видео: [Алексей Шипилёв — Java-объекты наизнанку](https://www.youtube.com/watch?v=q2wtSR3kD_I)
* Video: [Java Object Layout | Ordinary Object Pointer | Object Header | CompressedOops | OpenJdk HotSpot JVM](https://www.youtube.com/watch?v=MK4rcxpwiuc)
* Video: [JVM Benchmarking with Aleksey Shipilev](https://www.youtube.com/watch?v=x3Vlze1mUj4)


## What is layouter

Java VM and it's version.

HotSpot is from OpenJDK. The Lilliput in development.
The Raw is a layout by itself without real gaps and aligns.

CPU word size 32 or 64 bits i.e. size of a pointer.

COOPS is compressed references i.e. a trick to store 64 bits pointer in only 32 bits but all fields needs to be aligned.

Align is typically 8-byte but for a very large RAM may need to be 16-byte.

CCPS is Compressed Classes in an object header i.e. 4 bytes instead of 8.

You may find most typical layouters here https://github.com/openjdk/jol/blob/master/jol-cli/src/main/java/org/openjdk/jol/operations/EstimatedModels.java


## Related projects

Heap dump `*.hprof` files analysers:
 * [The Lightweight Java Visualizer (LJV)](https://github.com/atp-mipt/ljv) a tool for visualizing Java data structures 
 * The IntelliJ IDEA has a built-in [heap dump analyser](https://www.jetbrains.com/help/idea/analyze-hprof-memory-snapshots.html#read-snapshot)
 * Eclipse [Memory Analyzer (MAT)](https://www.eclipse.org/mat/)
 * [VisualVM](https://visualvm.github.io/) can also monitor heap in real time. Based on NetBeans
 * [Java Mission Control](https://github.com/openjdk/jmc)
 * [JMH Micro benchmarks tool](https://github.com/openjdk/jmh) and the [Intellij JMH plugin](https://github.com/artyushov/idea-jmh-plugin)