## Object Oriented Programming
There are at least three different approaches to object oriented programming in R. We examine two of them, the S3 and S4 classes. The other approach makes use of a package, and we focus instead on the two built in classes.

### Contents:
```
1. S3 Classes
2. S4 Classes
```
## S3 Classes

#### Contents
```
1. The Basic Idea
2. Memory Management
3. Creating an S3 Class
      3.1. Straight Forward Approach
      3.2. Local Environment Approach
4. Creating Methods
      4.1. Straight Forward Approach
      4.2. Local Environment Approach
5. Inheritance
```
We examine how to create S3 classes. It is assumed that you are familiar with the basic data types and scripting (Introduction to Programming).

### 1. The Basic Idea

First, everything in R is treated like as an object. We have seen this with functions. Many of the objects that are created within an R session have attributes associated with them. One common attribute associated with an object is its class.

You can set the class attribute using the class command. One thing to notice is that the class is a vector which allows an object to inherit from multiple classes, and it allows you to specify the order of inheritance for complex classes. You can also use the class command to determine the classes associated with an object.
```
> bubba <- c(1,2,3)
> bubba
[1] 1 2 3
>
> class(bubba)
[1] "numeric"
>
> class(bubba) <- append(class(bubba),"Flamboyancy")
> class(bubba)
[1] "numeric"     "Flamboyancy"
```
-----------------------------------------------------------------------------------------------------------------------
#### Note

A new command, append, is used here. The first argument is a vector, and the function adds the following arguments to the end of the vector.

----------------------------------------------------------------------------------------------------------------------- 

### METHOD 

One way to define a method for a class is to use the UseMethod command to define a hierarchy of functions that will react appropriately. The UseMethod command will tell R to look for a function whose prefix matches the current function, and it searches for a suffix in order from the vector of class names. In other words a set of functions can be defined, and the function that is actually called will be determined by the class name of the first object in the list of arguments.

You first have to define a generic function to reserve the function name. The UseMethod command is then used to tell the R system to search for a different function. The search is based on the name of the function and the names of an object’s classes. The name of the functions have two parts separated by a “.” where the prefix is the function name and the suffix is the name of a class.

#### That is a lot of verbiage to describe a relatively simple idea. A very basic example is given below:

```
> bubba <- list(first="one", second="two", third="third")
> class(bubba) <- append(class(bubba),"Flamboyancy")
>
> bubba
$first
[1] "one"

$second
[1] "two"

$third
[1] "third"

attr(,"class")
[1] "list"        "Flamboyancy"
>
> GetFirst <- function(x)
+ {
+     UseMethod("GetFirst",x)
+ }
>
> GetFirst.Flamboyancy <- function(x)
+ {
+    return(x$first)
+ }
>
> GetFirst(bubba)
[1] "one"
```
## 2. Memory Management

The plethora of object oriented approaches leads to a natural question. Which one should you use? With respect to S3 and S4 classes, the S3 class is more flexible, and the S4 class is a more structured approach. This is a nice way of saying that the S3 class approach is for unaware slobs and is a sloppy way to shoot yourself in the foot, while the S4 class is for uptight pedants.

Our focus here is on S3 classes. Before we delve into the details of S3 classes we need to talk about memory environments. These can be used to great effect in S3 classes to make your codes totally incomprehensible. On the down side they help give S3 classes their flexibility.

An environment can be thought of as a local scope. It has a set of variables associated with it. You can access those variables if you have the “ID’’ associated with the environment. There are a number of commands you can use to manipulate and obtain the pointers to your environments. You can also use the assign and get commands to set and get the values of variables within an environment.

### The environment command can be used to get the pointer to the current environment.
```
> ls()
character(0)
> e <- environment()
> e
<environment: R_GlobalEnv>
> assign("bubba",3,e)
> ls()
[1] "bubba" "e"
> bubba
[1] 3
> get("bubba",e)
[1] 3
```
