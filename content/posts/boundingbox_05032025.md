---
title: 2D Matrix Bounding Box Checker using DSU and Union-Find in Java 23 using TDD
subtitle: High level overview of using DSU / Union-Find Algorithm to efficiently detect bounding boxes in a 2D matrix using Java 23.
date: 2025-05-03T14:45:04-07:00
slug: 0cc3279
draft: false
author:
  name: Unnsse Khan
  link: https://github.com/unnsse
  email: contact@unnsse.io
  avatar:
description: High level overview of using DSU / Union-Find Algorithm to efficiently detect bounding boxes in a 2D matrix using Java 23.
keywords:  Unnsse Khan, untz, Java, Java Expert, Disjoint Set Union (DSU), Union-Find algorithm, Java 23, 2D Matrix, test driven development
license:
comment: false
weight: 0
tags:
  - java
  - java23
  - Disjoint Set
  - Union-Find
  - DSU
  - Algorithms
categories:
  - java
hiddenFromHomePage: false
hiddenFromSearch: false
hiddenFromRss: false
hiddenFromRelated: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
password:
message:
repost:
  enable: true
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

# Introduction

Using a Test Driven Development (TDD) approach, I wrote a Java 23 program called `BoundingBox` that reads 
a 2D ASCII grid from standard input and detects the largest or all non-overlapping bounding boxes enclosing 
contiguous regions of asterisks (`*`). It is designed to handle large inputs efficiently and uses a [Disjoint Set()
Union-Find)](https://en.wikipedia.org/wiki/Disjoint-set_data_structure) data structure algorithm to identify 
connected components. Each bounding box is defined by the minimum and maximum `x` and `y` coordinates 
(with 1-based indexing) that surround a connected group of * characters. 

Test-Driven Development was facilitated by reading prewritten input files from my unit tests. The goal was that the
`BoundingBox` would be able to accommodate all of these test cases. Lots of refactoring(s) took place as a result. The
final result was a well-organized and extensible codebase with comprehensive test coverage.

`BoundingBox` can be used for identifying and isolating distinct rectangular regions within a 2D grid,
making it useful in applications such as image processing, object detection, map region analysis, 
connected component labeling, and spatial clustering in large datasets or grid-based simulations.

# Features

• Detects all connected components of * characters using Union-Find.

• Computes the minimum bounding box for each component.

• Filters for the largest non-overlapping bounding box.

• Optionally returns all non-overlapping boxes in sorted order.

• Handles malformed input with a clear "Error" output.

• Efficient for large grids (10,000 × 10,000).

# Acceptance Criteria

This console app takes input from stdin with the following properties:

- Input is split into lines delimited by newline characters.

- Every line has the same length.

- Every line consists of an arbitrary sequence of hyphens `-` and asterisks `*`.

- The final line of input is terminated by a newline character.

Each character in the input will have coordinates defined by `(line number, character number)`,
starting at the top and left. So the first character on the first line will have the coordinates `(1,1)`
and the fifth character on line 3 will have the coordinates `(3,5)`.

The program should find a box (or boxes) in the input with the following properties:

- The box must be defined by two pairs of coordinates corresponding to its top left and bottom right corners.

- It must be the **minimum bounding box** for some contiguous group of asterisks, with each asterisk in the
  group being horizontally or vertically (but not diagonally) adjacent to each other. A single, detached asterisk
  is considered to be a valid box. The box should not _strictly_ bound the group, so the coordinates for the box
  in the following input should be `(2,2)(3,3)` not `(1,1)(4,4)`.
```txt
----
-**-
-**-
----
```

- It should not overlap (i.e. share any characters with) any other minimum bounding boxes.

e.g. Given this input:

```txt
--------
-******-
-*----*-
-*-**-*-
---**---
```

Result should be an empty string.

- Of all the non-overlapping, minimum bounding boxes in the input, _return the largest by area_.

If any boxes satisfying the conditions can be found in the input, the program should return an exit code
of 0 and, for each box, print a line to stdout with the two pairs of coordinates.

So, given the file “groups.txt” with the following content:
```
**-------***
-*--**--***-
-----***--**
-------***--
```

Running this program manually:
```
> ./bounding-box < groups.txt
```
Outputs:

```txt
(1,1)(2,2)
```

This is because the larger groups on the right of the input have overlapping bounding boxes,
so the returned coordinates bound the smaller group on the top left.

---

# Implementation

Decided to use a lot of new Java 23 features (records, var keyword, etc.) to write this program.
Didn't want nested for loops as they present a significant performance overhead when dealing with large input sets.
This was resolved using a lot of recursion along with `IntStream` functionality.

### Java 23's Records and IntStreams

Setting up the types as records was straightforward and also very concise and elegant.

Made a `Point` record storing x,y coordinates as integers and a `Box` record for using 
`Point` type as top left and bottom right corners of the box.

```java
public class BoundingBox {
    
    record Point(int x, int y) {
        @Override
        public String toString() {
            return "(" + x + "," + y + ")";
        }
    }
    
    record Box(Point topLeft, Point bottomRight) {
        long area() {
            return (long)(bottomRight.x() - topLeft.x() + 1) * (bottomRight.y() - topLeft.y() + 1);
        }
    
    @Override
    public String toString() {
        return topLeft.toString() + bottomRight.toString();
    }
    
    boolean overlaps(Box other) {
        return !(bottomRight.x() < other.topLeft.x() ||
                topLeft.x() > other.bottomRight.x() ||
                bottomRight.y() < other.topLeft.y() ||
                topLeft.y() > other.bottomRight.y());
    }
}
```
## overlaps() method

The `overlaps()` method was used to check if a box overlapped another box based on edge corners.

Logic Breakdown:

The method checks if none of the following conditions are true 
(using the `!` negation operator at the beginning):

1. If the current box is entirely to the left of the other box:

`bottomRight.x() < other.topLeft.x()`

Checks if the current box’s bottom-right corner is to the left of 
the other box’s top-left corner, implying no overlap.

2. If the current box is entirely to the right of the other box:

`topLeft.x() > other.bottomRight.x()`

Checks if the current box’s top-left corner is to the right of
the other box’s bottom-right corner, implying no overlap.

3. If the current box is entirely below the other box:

`bottomRight.y() < other.topLeft.y()`

Checks if the current box’s bottom-right corner is below the
other box’s top-left corner, implying no overlap.

4. If the current box is entirely above the other box:

`topLeft.y() > other.bottomRight.y()`

Checks if the current box’s top-left corner is above the other 
box’s bottom-right corner, implying no overlap.

If none of these conditions are true, the boxes overlap, so the method
returns true. Otherwise, it returns false.

## DSU / Union Find Algorithm

Implemented the DisjointSet data structure and Union Find algorithm as follows:

```java
static class DisjointSet {
    Map<Integer, Integer> parent = new HashMap<>();
    Map<Integer, Point> min = new HashMap<>();
    Map<Integer, Point> max = new HashMap<>();

    int find(int p) {
        parent.putIfAbsent(p, p);
        if (parent.get(p) != p) {
            parent.put(p, find(parent.get(p)));
        }
        return parent.get(p);
    }

    void union(int p1, int p2) {
        int r1 = find(p1), r2 = find(p2);
        if (r1 != r2) parent.put(r2, r1);
    }

    void updateBounds(int root, int x, int y) {
        min.merge(root, new Point(x, y),
                (v1, v2) -> new Point(Math.min(v1.x(), v2.x()), Math.min(v1.y(), v2.y())));
        max.merge(root, new Point(x, y),
                (v1, v2) -> new Point(Math.max(v1.x(), v2.x()), Math.max(v1.y(), v2.y())));
    }
}
```

This DisjointSet class implements a Union-Find (DSU) structure to manage connected components 
in a 2D grid. It uses a `Map<Integer, Integer>` to track the parent of each node for path compression 
in the `find()` method, ensuring efficient lookups. The union method merges two sets by assigning one root
as the parent of the other. Additionally, it maintains bounding box data for each component 
using min and max maps, which are updated via updateBounds to track the smallest and largest 
`Point` coordinates (x, y) associated with each set.

## findNonOverlappingBoxes() method

This method was written to extract a clean set of non-overlapping bounding boxes from a potentially overlapping 
collection, prioritizing consistency and simplicity for downstream testing or analysis.

```java
private List<Box> findNonOverlappingBoxes(List<Box> boxes) {
  // Sort boxes by topLeft.x, then topLeft.y for consistent ordering
  var sortedBoxes = boxes.stream()
          .sorted(Comparator.comparing((Box b) -> b.topLeft().x())
                  .thenComparing(b -> b.topLeft().y()))
          .toList();
  
  var nonOverlapping = new ArrayList<Box>();
  var used = new boolean[boxes.size()];
  
  IntStream.range(0, sortedBoxes.size())
          .filter(i -> !used[i])
          .forEach(i -> {
              nonOverlapping.add(sortedBoxes.get(i));
              used[i] = true;
              // Mark overlapping boxes as used
              IntStream.range(0, sortedBoxes.size())
                      .filter(j -> !used[j] && sortedBoxes.get(i).overlaps(sortedBoxes.get(j)))
                      .forEach(j -> used[j] = true);
          });
  
  return nonOverlapping;
}
```

The `findNonOverlappingBoxes()` method takes a list of `Box` objects and returns a subset containing
only the non-overlapping ones. It first sorts the boxes by their top-left `x` and `y` coordinates to 
ensure a consistent processing order. Then, it iterates through the sorted list, adding each unmarked 
box to the result list and marking it as used. For each box added, it also checks for and marks any 
other boxes that overlap with it, ensuring that overlapping boxes are excluded from further consideration. 
This approach guarantees that the returned list contains only distinct, non-overlapping boxes, favoring 
the earliest ones in the sorted order.

## Time and Space Complexity

|Operation | Time Complexity | Space Complexity |
| -------- | --------------- | ---------------- |
| Parsing and validation | O(n × m) | O(1) |
| Union-Find operations | O(α(n × m)) amortized | O(n × m) |
| Bounding box updates |O(k) | O(k) |
| Box overlap checks | O(k²) | O(k) |
| Final sorting & filtering | O(k log k) | O(k) |

Where:

• n = number of rows

• m = number of columns

• k = number of connected components (bounding boxes)

• α is the inverse Ackermann function (nearly constant in practice)

# Testing

All the data files for the numerous test cases were placed inside the test resources directory:

`BoundingBox/app/src/test/resources`

Wrote the following method to process each file per test case:

```java
private List<String> readInput(String resourceFileName) throws Exception {
    Path path = Path.of(Objects.requireNonNull(getClass().getClassLoader()
                    .getResource(resourceFileName))
            .toURI());
    return Files.readAllLines(path)
            .stream()
            .map(String::trim)
            .filter(line -> !line.isEmpty())
            .collect(Collectors.toList());
}
```
As one can ascertain, it reads all lines from the specified resource file, trims whitespace,
filters out empty lines, and returns the result as a list of strings — making it ideal for cleanly
loading test fixtures or input cases.

e.g. Using this to test `multiple-non-overlapping.txt`:

```txt
**---
**---
----*
----*
```
Was rather straightforward:

```java
@Test
  void testMultipleNonOverlapping() throws Exception {
      BoundingBox boundingBox = new BoundingBox();
      List<String> lines = readInput("multiple-non-overlapping.txt");
      assertEquals("(1,1)(2,2)", boundingBox.largestNonOverlappingBox(lines, false));
  }
```
# Running from Command Line

Since the acceptance criteria required BoundingBox to read input from stdin (e.g., via `./bounding-box < input.txt`),
I implemented this by checking the length of args and using a BufferedReader inside the main() method to read from
`System.in`:

```java
public static void main(String[] args) {
        if (args.length > 0) {
            System.err.println("Usage: ./bounding-box < input.txt");
            System.exit(1);
        }

        List<String> lines = new BufferedReader(new InputStreamReader(System.in))
                .lines()
                .map(String::trim)
                .filter(line -> !line.isEmpty())
                .collect(Collectors.toList());

        String result = new BoundingBox().largestNonOverlappingBox(lines, false); // Default: largest box
        System.out.println(result);
        System.exit(result.equals("Error") ? 1 : 0);
    }
```

# Afterthoughts

Very rewarding experience tackling all the various edge cases as presented in the unit tests. Being provided the input
test files and using a Test First approach really helped refine the codebase by tackling one unit test at time. The 
outcome was rather favorable with an extensible and organized codebase. The scalability of this program was also evident.
The DSU data structure with the Union-Find algorithm is essential for efficiently managing and merging disjoint sets, 
especially in problems involving connected components, clustering, network connectivity, image segmentation, 
and grid-based region detection.

# GitHub Repository
The full code is available here: https://github.com/unnsse/BoundingBox
