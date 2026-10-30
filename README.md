[![obsolete JetBrains project](https://jb.gg/badges/obsolete.svg)](https://confluence.jetbrains.com/display/ALL/JetBrains+on+GitHub)

⚠️ **This repository is no longer actively maintained.** ⚠️ 

Active Skija project is located at [HumbleUI/Skija](https://github.com/HumbleUI/Skija/)

# Skija: Java bindings for Skia

**Skia** is an open source 2D graphics library which provides common APIs that work across a variety of hardware and software platforms.

**Skija** is a high-quality Java bindings for Skia.

![](extras/logo.png)


<!-- fork-homepage-guide:begin -->
## Explore this historical fork

This fork preserves [JetBrains/skija](https://github.com/JetBrains/skija), the
Java/JNI bindings to Skia represented by this source tree. The maintenance warning
above matters: the upstream README directs new users to
[HumbleUI/Skija](https://github.com/HumbleUI/Skija/). This fork is useful for studying
the older API, native-resource ownership, raster drawing and platform integration;
it is not a claim that the historical JetBrains line has resumed maintenance.

At reviewed baseline
[`8581a6c`](https://github.com/hsoliwal/skija/commit/8581a6c04808c0ada7863aabed9f2a9d77353b39),
the default tree matched JetBrains upstream. This documentation adds a first
experiment, not a new renderer or verified performance improvement. Historical
platform/backend lists and benchmark wording below retain their upstream context;
qualify the exact checkout on your target OS and architecture.

### Five drawing concepts

| Concept | Role | Source |
| --- | --- | --- |
| Surface | Owns a drawing destination, such as an in-memory raster | [Surface.java](shared/java/Surface.java) |
| Canvas | Issues drawing commands to a destination | [Canvas.java](shared/java/Canvas.java) |
| Paint | Color, stroke and effects used by a drawing call | [Paint.java](shared/java/Paint.java) |
| Image / Data | A snapshot and its encoded bytes | [Image.java](shared/java/Image.java), [Data.java](shared/java/Data.java) |
| Managed resource | A Java wrapper whose lifetime controls native memory | [Managed.java](shared/java/impl/Managed.java) |

A canvas obtained from `surface.getCanvas()` is managed by that surface; do not
close it independently. Close resources you own after their last use. A Java heap
measurement alone does not describe native Skia memory.

### Quickstart: build and inspect the local bindings

Use Git, Python 3, JDK 11+ with `JAVA_HOME` and matching Java tools on `PATH`, CMake,
Ninja and a platform C++ toolchain. The scripts download their pinned historical
Skia archive and Java dependencies; availability and host compatibility must be
checked rather than inferred from today's Skia releases.

```sh
git clone https://github.com/hsoliwal/skija.git
cd skija
git rev-parse HEAD
java -version
python3 script/build.py
python3 script/test.py
```

The [build script](script/build.py) places Java/platform resources under
`shared/target/classes` and `platform/target/classes`, with native build products
under `platform/build`. The [test script](script/test.py) builds as needed, enables
assertions and JNI checks, then runs `org.jetbrains.skija.test.TestSuite`.
Passing that suite does not qualify all UI frameworks or GPU backends.

### Worked example: a red circle without a window

After configuring a matching platform dependency using [Using Skija](#using-skija)
or your local build, place this method in a Java class with these imports:

```java
import org.jetbrains.skija.*;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;

static void writeCircle() throws IOException {
    try (Surface surface = Surface.makeRasterN32Premul(100, 100);
         Paint paint = new Paint().setColor(0xFFFF0000)) {
        Canvas canvas = surface.getCanvas();
        canvas.clear(0xFFFFFFFF);
        canvas.drawCircle(50, 50, 30, paint);
        try (Image image = surface.makeImageSnapshot();
             Data png = image.encodeToData(EncodedImageFormat.PNG)) {
            if (png == null) throw new IOException("PNG encoding failed");
            Files.write(Path.of("skija-circle.png"), png.getBytes(),
                StandardOpenOption.CREATE_NEW, StandardOpenOption.WRITE);
        }
    }
}
```

This example method uses JDK 11+ (`Path.of`). It is source-checked, not executed
by this documentation update. The expected output is a 100×100 PNG with a red
circle on white. `CREATE_NEW` refuses to overwrite an existing file. The raster
path needs the matching native library but no window or GPU context; a successful
raster example is not evidence for OpenGL, Direct3D or Metal rendering.

### Troubleshooting and reading cheat sheet

| Question | First place to look |
| --- | --- |
| Native library fails to load | Match OS/architecture, Java module path and platform resources; inspect [platform/](platform/) |
| Wrong alpha/color | Colors here are ARGB; `0xFF` alpha is opaque |
| Native memory lasts too long | Follow resource ownership and `AutoCloseable` scopes |
| Need a window or event loop | [LWJGL examples](examples/lwjgl/) and [Getting Started](docs/Getting%20Started.md) |
| Need typography and API conventions | [Getting Started](docs/Getting%20Started.md), [Conventions](docs/Conventions.md), [Java source](shared/java/) |
| Need current project development | Follow the maintained-project link above and review its own API/build instructions |

Retain [LICENSE](LICENSE) and [NOTICE](NOTICE) when reusing this tree. Report the
exact revision, OS/architecture, JDK, native library and smallest drawing example;
do not mix newer Skia binaries into this pinned historical build without validation.

<!-- fork-homepage-guide:end -->

## Motivation: Why Skija?

Because Java doesn’t have a powerful and modern 2D graphics — still!

There are many occasions when you might need graphics:

- custom UI widget libraries and whole toolkits,
- graphs, diagrams,
- visualizations,
- games.

Existing offerings are: Graphics2D from AWT, GraphicsContext from JavaFX. They are good, but underwhelming.

Enter Skia. Skia has a proven track record of industrial-scale project relying on it for all things graphics: Google Chrome, Android, Flutter, Firefox Canvas, Xamarin, LibreOffice. Skia outperforms all existing Java2D toolkits in almost every benchmark, and also provides:

- extensive color spaces support,
- modern typography with open type features, variable typefaces, correct multi-script text handling, emojis,
- highly-optimized GPU rendering,
- modern GPU backends, including Vulkan and Metal (already in Skia, coming to Skija soon),
- built-in caching and compositing facilities.

Skija brings all this power to the tips of your fingers, as simple as adding a jar to classpath.

## Why hand-crafted bindings?

Automatically generated bindings for Skia exist, but don’t seem to have high adoption:

- [github.com/bytedeco/javacpp-presets/tree/master/skia](https://github.com/bytedeco/javacpp-presets/tree/master/skia)
- [github.com/eungju/skia-javacpp](https://github.com/eungju/skia-javacpp)

Skija project has a goal of providing great Java-native API that is natural to use. In particular:

- full automatic memory management, no pointer abstractions leaking,
- natural use of Java classes, interfaces, inheritance, singletons,
- consistent naming following Java conventions, including getters/setters for properties,
- typed enums instead of integer constants,
- native Java platform abstractions instead of wrapped Skia/C++ ones (strings, arrays, streams, files, byte buffers, AutoCloseable),
- hiding implementation details, e.g. transparent string encoding conversion, byte/code point indices conversion,
- fluent builder-style APIs where possible,
- lightweight data classes where possible (Point, Rect, FontMetrics, etc are not mirrored by native instances).

The ultimate goal for Skija is to feel as a Java library and not having to think about native part at all.

## Built with Skija

[Compose for Desktop](https://www.jetbrains.com/lp/compose/), declarative UI toolkit for Kotlin:

![](extras/compose-apps.png)

[Skija Graphics2D](https://github.com/jfree/skijagraphics2d/), an implementation of Java2D API:

![](extras/skijagraphics2d.png)

[Robert Felker](https://twitter.com/BlueAquilae/status/1322696020329902082) demos:

|![](extras/Elso-W-XYAEKmFg.jpg)|![](extras/Ev-CCy_XMAEM51f.jpg)|
|---|---|
|![](extras/Elso-WsWoAExiUF.jpg)|![](extras/Elso-WfWMAED8dc.jpg)|

[Harold](https://www.instagram.com/_hrrld/) videos:

![](extras/110151060-3a840480-7d9d-11eb-9996-a8dc1ed1ab6a.jpg)

[Skija demo app](examples/lwjgl):

|![](extras/geometry.png)|![](extras/text.png)|
|---|---|
|![](extras/bitmap.png)|![](extras/pythagoras.png)|
|![](extras/filters.jpg)|![](extras/shaders.png)|

## Current status

Public alpha. Things might change without notice.

Please note that Skia is known to change its API quite often (monthly). Skija will do its best to protect from these changes, but sometimes it might be impossible (e.g. method was removed). Normally it’s auxiliary stuff though, don’t worry too much.

Platforms:

- [x] Windows
- [x] Linux
- [x] macOS

Backends:

- [x] Bitmap
- [x] OpenGL
- [x] Direct3D
- [x] Metal
- [ ] Vulkan

APIs:

```
Bitmap               ▓▓▓▓▓▓▓▓▓▓    Paint                ▓▓▓▓▓▓▓▓▓▓
Canvas               ▓▓▓▓▓▓▓▓░░    Path                 ▓▓▓▓▓▓▓▓▓▓
Codec                ▓▓▓▓░░░░░░    PathEffects          ▓▓▓▓▓▓▓▓▓▓
Color                ▓░░░░░░░░░    PathMeasure          ▓▓▓▓▓▓▓▓▓▓
ColorFilter          ▓▓▓▓▓▓▓▓▓▓    PaintFilterCanvas    ▓▓▓▓▓▓▓▓▓▓
ColorInfo            ▓▓▓▓▓▓▓▓▓▓    Picture              ▓▓▓▓▓▓▓▓▓░
ColorSpace           ▓▓▓▓░░░░░░    PictureRecorder      ▓▓▓▓▓▓▓▓▓▓
Data                 ▓▓▓▓▓▓▓▓▓░    PixelRef             ▓▓▓▓▓▓▓▓▓▓
Drawable             ▓▓▓▓▓▓▓▓░░    Pixmap               ▓▓▓▓▓▓▓▓▓▓
Flattenable          ░░░░░░░░░░    Region               ▓▓▓▓▓▓▓▓▓▓
Font                 ▓▓▓▓▓▓▓▓▓▓    RuntimeEffect        ▓▓▓▓▓░░░░░
FontData             ░░░░░░░░░░    ScalerContext        ░░░░░░░░░░
FontManager          ▓▓▓▓▓▓▓▓▓░    Shader               ▓▓▓▓▓▓▓▓▓▓
FontStyle            ▓▓▓▓▓▓▓▓▓▓    ShadowUtils          ▓▓▓▓▓▓▓▓▓▓
FontStyleSet         ▓▓▓▓▓▓▓▓▓▓    Stream               ░░░░░░░░░░
Image                ▓▓░░░░░░░░    String               ▓░░░░░░░░░
ImageFilters         ▓▓▓▓▓▓▓▓▓▓    Surface              ▓░░░░░░░░░
ImageInfo            ▓▓▓▓▓▓▓▓▓▓    TextBlob             ▓▓▓▓▓▓▓▓▓▓
MaskFilter           ▓▓▓▓▓▓▓▓▓▓    TextBlobBuilder      ▓▓▓▓▓▓▓▓▓▓
Matrix33             ▓▓▓░░░░░░░    Typeface             ▓▓▓▓▓▓▓▓░░
Matrix44             ▓▓▓░░░░░░░    WStream              ▓▓░░░░░░░░

Shaper:                            Paragraph:

BiDiRunIterator      ▓▓▓▓▓▓▓▓▓▓    FontCollection       ▓▓▓▓▓▓▓▓▓▓
FontMgrRunIterator   ▓▓▓▓▓▓▓▓▓▓    LineMetrics          ▓▓▓▓▓▓▓▓▓░
FontRunIterator      ▓▓▓▓▓▓▓▓▓▓    Paragraph            ▓▓▓▓▓▓▓▓▓▓
HbIcuScriptRunIter   ▓▓▓▓▓▓▓▓▓▓    ParagraphCache       ▓▓▓▓▓▓▓▓▓▓
IcuBidiRunIterator   ▓▓▓▓▓▓▓▓▓▓    ParagraphStyle       ▓▓▓▓▓▓▓▓▓▓
LanguageRunIterator  ▓▓▓▓▓▓▓▓▓▓    ParagraphBuilder     ▓▓▓▓▓▓▓▓▓▓
RunHandler           ▓▓▓▓▓▓▓▓▓▓    TextStyle            ▓▓▓▓▓▓▓▓▓▓
RunInfo              ▓▓▓▓▓▓▓▓▓▓    TypefaceFontProvider ▓▓▓▓▓▓▓▓▓▓
ScriptRunIterator    ▓▓▓▓▓▓▓▓▓▓
Shaper               ▓▓▓▓▓▓▓▓▓▓
TextBlobBldRunHndlr  ▓▓▓▓▓▓▓▓▓▓

SVG:

SVGDOM               ▓▓▓▓▓▓▓▓░░
SVGCanvas            ▓▓▓▓▓▓▓▓▓▓
SVGSVG               ▓▓▓▓▓▓▓▓░░
```

## Using Skija

Using Skija is as simple as adding a jar file.

Maven:

```xml
<repositories>
  <repository>
    <id>space-maven</id>
    <url>https://packages.jetbrains.team/maven/p/skija/maven</url>
  </repository>
</repositories>

<dependencies>
  <dependency>
    <groupId>org.jetbrains.skija</groupId>
    <artifactId>skija-${platform}</artifactId>
    <version>${version}</version>
  </dependency>
</dependencies>
```

Gradle:

```gradle
repositories {
  maven {
    url "https://packages.jetbrains.team/maven/p/skija/maven"
  }
}

dependencies {
  api "org.jetbrains.skija:${artifact}:${version}"
}
```

Replace `${artifact}` and `${version}` with:

Platform    | `${artifact}`       | `${version}`
------------|---------------------|-------------
macOS Intel | `skija-macos-x64`   | ![version](https://img.shields.io/badge/dynamic/xml?style=flat-square&label=latest&color=success&url=https%3A%2F%2Fpackages.jetbrains.team%2Fmaven%2Fp%2Fskija%2Fmaven%2Forg%2Fjetbrains%2Fskija%2Fskija-macos-x64%2Fmaven-metadata.xml&query=//release)
macOS M1    | `skija-macos-arm64` | ![version](https://img.shields.io/badge/dynamic/xml?style=flat-square&label=latest&color=success&url=https%3A%2F%2Fpackages.jetbrains.team%2Fmaven%2Fp%2Fskija%2Fmaven%2Forg%2Fjetbrains%2Fskija%2Fskija-macos-arm64%2Fmaven-metadata.xml&query=//release)
Linux       | `skija-linux`       | ![version](https://img.shields.io/badge/dynamic/xml?style=flat-square&label=latest&color=success&url=https%3A%2F%2Fpackages.jetbrains.team%2Fmaven%2Fp%2Fskija%2Fmaven%2Forg%2Fjetbrains%2Fskija%2Fskija-linux%2Fmaven-metadata.xml&query=//release)
Windows     | `skija-windows`     | ![version](https://img.shields.io/badge/dynamic/xml?style=flat-square&label=latest&color=success&url=https%3A%2F%2Fpackages.jetbrains.team%2Fmaven%2Fp%2Fskija%2Fmaven%2Forg%2Fjetbrains%2Fskija%2Fskija-windows%2Fmaven-metadata.xml&query=//release)

## Documentation

Get started by reading [Getting Started](docs/Getting%20Started.md).

API docs are under development — consult source code and JavaDoc comments.

I found [SkiaSharp documentation](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/graphics/skiasharp/) to be excellent resource on what can be done in Skia. They have nice examples and visual explanations, too.

If Skija is missing a documentation for a particular method or class, check the same class in [Skia Documentation](https://api.skia.org/) instead. It might be that we didn’t move it over to Java yet. PRs are welcome!

Finally, [LWJGL demo app](/examples/lwjgl) has examples of most of the APIs that are currently implemented.

## Resources

- Overview [youtube.com/watch?v=t1X-Oln1u24](https://www.youtube.com/watch?v=t1X-Oln1u24)
- Coding game in Skija and Clojure [youtube.com/watch?v=KES-lKTq-3M](https://www.youtube.com/watch?v=KES-lKTq-3M)
- SWT and Skija on Raspberry Pi [spket.com/blog/native-gui-app-with-swt-and-skija-on-raspberry-pi](https://www.spket.com/blog/native-gui-app-with-swt-and-skija-on-raspberry-pi/)

## Building Skija from scratch

Prerequisites: Git, CMake, Ninja, JDK 9+, $JAVA_HOME, Python 3.

Checkout:

```sh
git clone https://github.com/JetBrains/skija.git
cd skija
./script/build.py
```

To codesign:

```sh
security find-identity
export APPLE_CODESIGN_IDENTITY="<...>"
./script/build.py
```

For building Skia itself, see https://github.com/JetBrains/skia-build/

## Running examples

Examples require local build of Skija (see [Building Skija](#building-skija-from-scratch)).

See [examples/](examples/).

## Contributing

Contributions are welcome!

Make sure to read [Code Conventions](docs/Conventions.md).

Remember: the goal of Skija is to map Skia API as close as possible.

These things have a place in Skija:

- Everything that is in Skia and makes sense in Java world.
- Convenience methods (e.g. a method that uses a default value for a parameter).
- Documentation on Skija methods is very welcome!

These don’t:

- Things that don’t directly map to anything in Java (pointers, etc).
- New classes, new methods that don’t exist in Skia.
- Code that combines Skia APIs together for special use-cases.

Useful things built on top of Skia/Skija are always welcome — as libraries.
