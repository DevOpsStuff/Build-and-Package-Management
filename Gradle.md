# Gradle Build Tool

## Introduction

### What is Gradle

* Build by convention
* Written in Groovy, it's a Groovy DSL
* Dependency management with Ivy or Maven
* Supports multi-project builds
* Can build things other than Java projects such as JavaScript or C++
* Highly customizable
* Declarative Build Language (expresses intent)
  * We tell Gradle _what_ we would like to happen, not _how_
  * Makes build scripts easier to read than Ant or Maven xml
  * Maintainable
  
### Comparsion with other Java based Build tools
 
 * Ant (Another Neat tool)
      * XML Build Script
          * Hard to read
          * Bit Difficult 
 * Maven
      * Supports dependencies
      * Highly Extensible
      * It is written in xml
  
## Install Gradle

   * From the gradle website https://gradle.org/install/ . Create a directory under c:/Gradle after extracting the gradle move it to the c:\Gradle. Add the gradle\bin path to the windows env path. And reload the command prompt.
   Now type, `gradle -version`.
   
   * For Linux, get from gvmtool. `curl -s get.gvm.io | bash` and run `gvm install gradle`.
      
### Gradle Builds

* Build file is typically named `build.gradle`
* Build file contains _tasks_, and
  * plugins
  * dependencies
  
### simple Gradle file

  * create a build.gradle file

       ```
            task hello {
             doLast {
                println "Hello, World"
               }
            }
       ```
 
#### Build Java

        ```groovy
        
           apply plugin: 'java'
           
        ```

  The java plugin uses a convention for directory structure

        ```
        ├── example
        │   ├── build.gradle
        │   └── src
        │       ├── main
        │       │   └── java
        │       │       └── learning
        │       │           └── App.java
        │       └── test
        │           └── java
        │               └── learning
        │                   └── AppTest.java
        
        ```

Running `gradle tasks` displays a list of all the tasks that the java plugin has installed.

Run `gradle build` to build the project.

Gradle output will mark a task `UP-TO-DATE` if there's nothing to do. Re-running a build will only rebuild things that have changed.

Gradle output goes to the `build` folder, example compiled classes, libs, jar files, etc.

   
### What is a Task

* Code that Gradle will execute
* Has a __lifecycle__ (different parts of the task will run at different times)
  * Initialization phase
  * Configuration phase
  * Execution phase
* Has __properties__
  * Common properties such as description, and group it belongs to
  * Custom to that task such as directory files are being copied to or from
  * Properties are typically configured during the Configuration Phase of the lifecycle
* Has __actions__ (code that executes), divided into two parts:
  * First action - code that needs to run before other code within the task
  * Last action - code that executes as part of the task
* Has __dependencies__: A task may require that another task complete before this one can execute. Gradle will work out the task dependencies and take care that tasks run in the correct order.

### Writing Simple Tasks

Groovy is object oriented. In a Gradle build script, top level object is `project`.
This object is used to define everything within build script.

To create a task:

      ```groovy
         project.task("Task1")
      ```

To list all tasks

      ```shell
        $ gradle tasks
      ```

Notice "Task1" is in a group called "Other tasks".

Can also define a task without `project` keyword, because Gradle knows "project" is the top level object and will delegate everything to it:

      ```groovy
         task("Task2")
      ``` 

Another way to define a task, don't need brackets:

      ```groovy
         task "Task3"
      ```

Finally, don't even need the quotes to define a task:

      ```groovy
         task Task4
      ```

To add a property to a task:

       ```groovy
           Task4.description = "My super duper awesome task"
        ```

Description will appear in shell when running `gradle tasks`.

### Running Tasks

First need to write a task action. All tasks have a `doLast` method, which is the last thing the task does. It gets passed a _Groovy closure_, which is code that lives between braces.

      ```groovy
         task Task4
         Task4.description = "My super duper awesome task"
         Task4.doLast { println "This is Task 4" }
      ```

To run the task:

      ```shell
        $ gradle Task4
      ```

Another way to write a task action is to use left shift operator `<<` instead of explicit `doLast`.
`<<` is overridden by Groovy to add a closure to `doLast`.

      ```groovy
         task Task3
          Task3 << println "This is Task 3"
      ```

Can declare a task and add action in one line:

      ```groovy
         task Task5 << { println "This is Task 5" }
      ```

Note you can keep adding clojures, and Gradle will append them:

      ```groovy
         task Task5 << { println "This is Task 5" }
         Task5 << { println "Another closure" }
      ```

Can add properties and actions all at once:

      ```groovy
         task Task6 {
         description "Task 6 is the best task ever"
         doLast {
              println "Task 6 is running"
            }
          }
      ```

### Task Phases

* Initialization phase: Used to configure multi project builds
* Configuration phase: Executes code in the task that's not in the action, for example, setting the "description" property
* Execution phase: Execute the task actions such as `doFirst`, `doLast`

Every task has a `doFirst` method, for example:

         ```groovy
            task Task6 {
            description "Task 6 is the best task ever"
            doFirst {
                "Task 6 first"
                }
            doLast {
                 println "Task 6 is running"
                }
            }
          ```

A task can have multiple `doFirst`'s and multiple `doLast`'s. When left shift `<<` operator is used to append a task, it gets added to the `doLast`. To add multiple `doFirst`'s, simply call it multiple times:

          ```groovy
              task Task6 {
                    description "Task 6 is the best task ever"
                    doFirst {
                          println "Task 6 first"
                    }
                    doLast {
                          println "Task 6 is running"
                    }
              }

              Task6.doFirst {
                    println "Task 6 another doFirst closure was appended"
              }
            ```

Output from running `gradle Task6`:

            ```
            :Task6
             Task 6 another doFirst closure was appended
             Task 6 first
             Task 6 is running

             BUILD SUCCESSFUL
              ```

### Task Dependencies

Each task has a `dependsOn` method that can be passed the tasks that this task depends on:

          ```groovy
            Task6.dependsOn Task5
          ```

Now running Task6 will FIRST run Task5, then Task6. Notice that Task6's `doFirst` methods run _after_ Task5's `doLast` methods:

          ```
            Task5
            This is Task 5
            Another closure
            :Task6
            Task 6 another doFirst closure was appended
            Task 6 first 
            Task 6 is running
          ```

Continuing on, if Task5 depends on Task4, then running Task6 will first run 4, 5, then 6.

A task can also have _multiple_ dependencies:

          ```groovy
              Task6.dependsOn Task5
              Task6.dependsOn Task3
          ```

Can also specify multiple dependencies as comma separated list:

          ```groovy
             Task6.dependsOn Task5, Task3
          ```

Can also specify dependencies inside the task closure:

          ```groovy
             task Task7 {
                description "This is task 7"
                dependsOn Task6
                doFirst {
                      println "Task 7 doFirst"
                }
                doLast {
                      println "Task 7 doLast"
                }
          }
          ```

Dependencies are built during the configuration phase. When a task is executed, the dependency graph has already been determined, and Gradle can simply walk through the graph, executing tasks.

### Setting Properties on Tasks

Properties can be defined using local variables with `def` keyword.
Then the variable can be used in tasks using string interpolation.

Local variable has scope of the given build file in which its declared. This might not be suitable for a multi-project build if need the variable available in other build files.

Variables can also be defined inside a task, in a closure.

          ```groovy
             def projectVersion = "2.1.0"

             Task6 {
                  doLast {
                        println "This is task 6 - version $projectVersion"
                  }
             }
          ```

To have a variable available in a larger scope, across projects, use _extra properties_.

        ```groovy
           project.ext.projectVersion = "2.1.0"
           ...
          doFirst {
              println "Task 6 first - some big project var $project.ext.biggerScopeVariable"
          }
        ```

Because its set on project, don't need to state it explicitly, so can just say:

        ```groovy
            ext.projectVersion = "2.1.0"
            ...
            doFirst {
                  println "Task 6 first - some big project var $biggerScopeVariable"
            }
        ```
