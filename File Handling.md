# 📁 Java File Handling

* File handling refers to the process of creating, opening, reading, writing, and closing files within a program, allowing data to be stored permanently on a computer's secondary storage (HDD/SSD) rather than being lost when the program terminates. 
*  It manages the flow of data between a program and the filesystem, essential for data persistence, managing large datasets, and maintaining configurations

Here we can see how **file handling works in Java**, including:

* Core concepts
* Classic I/O vs Modern NIO
* Important classes (what, why, how)
* Best practices
* Full working examples

---

## 📌 Table of Contents

* [Why File Handling Matters](#why-file-handling-matters)
* [Java File Handling APIs](#java-file-handling-apis)
* [Core java.io Classes](#core-javaio-classes)
* [Modern NIO System](#modern-nio-system)
* [How It Works Internally](#how-it-works-internally)
* [Common Exceptions](#common-exceptions)
* [Best Practices](#best-practices)
* [Full Examples](#full-examples)
* [When To Use What](#when-to-use-what)
* [Handling Images in Java](#handling-images-in-java)

---

## 🎯 Why File Handling Matters

Programs normally lose data when they stop running.

File handling allows you to:

* Save user data permanently
* Load configuration files
* Store logs
* Process large datasets
* Read/write reports

---

## 🧱 Java File Handling APIs

Java provides two systems:

### 1️⃣ Classic I/O → `java.io`

* Stream based
* Older but fundamental
* Still widely used

### 2️⃣ Modern I/O → `java.nio.file`

* Faster
* Cleaner
* Handles large files efficiently

---

# 📦 Core java.io Classes

## 📄 File (Path Representation)

```java
File file = new File("data.txt");
```

### ✅ What it does

Represents file or folder path — NOT file content.

### 🎯 Why it exists

* Check file existence
* Create/delete files
* Get metadata

### 🔧 Common methods

```java
file.exists();
file.createNewFile();
file.delete();
file.getName();
file.length();
```

---

## 📥 FileInputStream (Binary Reading)

```java
FileInputStream fis = new FileInputStream("data.txt");
```

Reads raw bytes from file.

### Used for:

* Images
* Audio/video
* Binary data

---

## 📤 FileOutputStream (Binary Writing)

```java
FileOutputStream fos = new FileOutputStream("data.txt");
```

Writes bytes to file.

---

## 📝 FileReader (Text Reading)

```java
FileReader fr = new FileReader("data.txt");
```

Reads characters (better for text files).

---

## ✍ FileWriter (Text Writing)

```java
FileWriter fw = new FileWriter("data.txt");
```

Writes characters.

---

## 🚀 BufferedReader (Fast Text Reading)

```java
BufferedReader br = new BufferedReader(new FileReader("data.txt"));
```

Reads data in chunks instead of one char at a time.

```java
br.readLine();
```

---

## ⚡ BufferedWriter (Fast Text Writing)

```java
BufferedWriter bw = new BufferedWriter(new FileWriter("data.txt"));
```

Improves performance.

---

# 🌐 Modern NIO System

Package:

```java
java.nio.file
```

---

## 📍 Path

```java
Path path = Paths.get("data.txt");
```

Modern file location object.

---

## 🧰 Files (Utility Class)

```java
Files.createFile(path);
Files.write(path, data);
Files.readAllLines(path);
Files.delete(path);
```

### Why better:

* One line operations
* Auto resource handling
* High performance

---

# 🧠 How It Works Internally

```text
Java Program
   ↓
Stream / Buffer
   ↓
Operating System
   ↓
Disk File
```

1. Java requests OS
2. OS opens file buffer
3. Data flows via stream
4. File closed → memory freed

---

# ⚠ Common Exceptions

| Exception             | Meaning           |
| --------------------- | ----------------- |
| FileNotFoundException | File missing      |
| IOException           | Read/write error  |
| SecurityException     | Permission denied |

---

# ✅ Best Practices

* Always close streams
* Use try-with-resources
* Prefer Buffered classes
* Use NIO for big files
* Handle exceptions properly

---

# 📘 Full Example (Classic I/O)

```java
import java.io.*;

public class FileExample {
    public static void main(String[] args) {
        try {
            FileWriter fw = new FileWriter("data.txt");
            fw.write("Hello Java File Handling!\nWelcome to Files.");
            fw.close();

            BufferedReader br = new BufferedReader(new FileReader("data.txt"));
            String line;

            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }

            br.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

# ⚡ Modern Example (NIO)

```java
import java.nio.file.*;
import java.io.IOException;
import java.util.*;

public class NIOExample {
    public static void main(String[] args) throws IOException {
        Path path = Paths.get("data.txt");

        List<String> data = Arrays.asList(
            "Hello NIO",
            "Fast and clean file handling"
        );

        Files.write(path, data);

        List<String> lines = Files.readAllLines(path);

        for (String line : lines) {
            System.out.println(line);
        }
    }
}
```

---

# 📊 When To Use What

| Task            | Best Tool             |
| --------------- | --------------------- |
| Simple text     | BufferedReader/Writer |
| Large files     | NIO Files             |
| Binary          | FileInputStream       |
| Fast operations | NIO                   |

---

# 🖼 Handling Images in Java

Java provides built‑in support for reading, writing, and manipulating images using the **ImageIO API**.

Main package:

```java
javax.imageio
```

---

## 📦 Core Image Classes

### 🧱 BufferedImage (Image in memory)

```java
BufferedImage image;
```

### What it does

Stores an image as pixels in RAM.

### Why it’s used

* Modify pixels
* Resize
* Convert formats
* Analyze colors

---

### 📥 ImageIO (Read/Write Engine)

```java
ImageIO.read(file);
ImageIO.write(image, "png", file);
```

Handles encoding/decoding of image formats.

Supports:

* PNG
* JPG
* BMP
* GIF

---

# 📖 Reading an Image

```java
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.File;

public class ReadImage {
    public static void main(String[] args) throws Exception {
        File file = new File("photo.jpg");
        BufferedImage image = ImageIO.read(file);

        System.out.println("Width: " + image.getWidth());
        System.out.println("Height: " + image.getHeight());
    }
}
```

---

# 💾 Writing an Image

```java
ImageIO.write(image, "png", new File("output.png"));
```

---

# 🎨 Accessing Pixels

```java
int pixel = image.getRGB(x, y);
```

Extract color components:

```java
int r = (pixel >> 16) & 0xff;
int g = (pixel >> 8) & 0xff;
int b = pixel & 0xff;
```

---

# 🔄 Modifying Image (Grayscale Example)

```java
for (int y = 0; y < image.getHeight(); y++) {
    for (int x = 0; x < image.getWidth(); x++) {
        int p = image.getRGB(x, y);

        int r = (p >> 16) & 0xff;
        int g = (p >> 8) & 0xff;
        int b = p & 0xff;

        int gray = (r + g + b) / 3;

        int newPixel = (gray << 16) | (gray << 8) | gray;
        image.setRGB(x, y, newPixel);
    }
}

ImageIO.write(image, "jpg", new File("gray.jpg"));
```

---

# 🚀 Image Handling with Streams

```java
FileInputStream fis = new FileInputStream("photo.png");
BufferedImage img = ImageIO.read(fis);
```

Works with network images, uploads, binary streams.

---

# 📊 Common Image Operations

| Task         | Method          |
| ------------ | --------------- |
| Load image   | ImageIO.read()  |
| Save image   | ImageIO.write() |
| Get pixel    | getRGB()        |
| Change pixel | setRGB()        |
| Resize       | Graphics2D      |

---

# ⚠ Common Issues

* NullPointerException → file not found
* Unsupported format
* Large image memory usage

---

# ✅ Best Practices for Images

* Use PNG for quality
* Use JPG for smaller size
* Close streams
* Avoid loading massive images fully
* Prefer BufferedImage

---

# 🏁 Final Summary

Java file handling works using:

Streams → Buffers → OS File System

Modern Java prefers:

✅ Path + Files (NIO)

Classic I/O builds strong fundamentals.

---
