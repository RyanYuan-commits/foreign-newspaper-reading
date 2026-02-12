---
source: https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html
---
## Overview

### Purpose

This tutorial covers **the basics of how to use the G1 garbage collector** and **how it can be used with the Hotspot JVM**. You will learn how the G1 collector functions [[internally]], the key command line switches for using G1, and options for logging its operation.

### Introduction

This OBE covers the basics of Java Virtual Machine(JVM) G1 Garbage Collection (GC) in Java. In the first part of the OBE, an overview of the JVM is provided along with an introduction to Garbage Collection and performance. Next students are provided with a review of how the CMS collector works with the Hotspot JVM. Next, a step by step guide to how Garbage Collection works when using the G1 garbage collection with a Hotspot JVM. Following that, a section is provided covering the Garbage Collection command line options available with the G1 garbage collector. Finally, you will learn about logging options to use with the G1 collector.

### Hardware and Software Requirements

The following is a list of hardware and software requirements:

- A PC running Windows XP or later, Mac OS X or Linux. Note that the hands on is done with Windows 7 and has not been tested on all platforms. However, everything should work fine on OS X or Linux. Also a machine with more than one core is preferable.
- Java 7 Update 9 or later
- The latest Java 7 Demos and Samples Zip file

### Prerequisites

Before starting this tutorial, you should:

- If you have not done so, download and install the latest version of the Java JDK (JDK 7 u9 or later).  
	
- Download and install the Demos and Samples zip file from the same location. Unzip the file and place the contents in a directory. For example: `C:\javademos`