## Object Oriented Programming
There are at least three different approaches to **object oriented programming** in R. We examine two of them, the S3 and S4 classes. The other approach makes use of a package, and we focus instead on the two built in classes.

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

First, **everything in R is treated like as an object**. We have seen this with functions. Many of the objects that are created within an R session have attributes associated with them. **One common attribute associated with an object is its class**.

You can **set the class attribute using the class** command. One thing to notice is that the **class is a vector which allows an object to inherit from multiple classes**, and it allows you to specify the order of inheritance for complex classes. You can also use the class command to determine the classes associated with an object.
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

A new command, **append**, is used here. **The first argument is a vector, and the function adds the following arguments to the end of the vector.

----------------------------------------------------------------------------------------------------------------------- 

### METHOD 

One way to define a method for a class is to use the **UseMethod** command to define a hierarchy of functions that will react appropriately. The **UseMethod command will tell R to look for a function whose prefix matches the current function, and it searches for a suffix in order from the vector of class names**. In other words a set of functions can be defined, and the function that is actually called will be determined by the class name of the first object in the list of arguments.

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

The plethora of object oriented approaches leads to a natural question. **Which one should you use? With respect to S3 and S4 classes, the S3 class is more flexible, and the S4 class is a more structured** approach. This is a nice way of saying that the S3 class approach is for unaware slobs and is a sloppy way to shoot yourself in the foot, while the S4 class is for uptight pedants.

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
Environments can be created and embedded within other environments and can be structured to form a hierarchy. 

## 3. Creating an S3 Class

The basic ideas associated with S3 classes is discussed in the first section (The Basic Idea). We now expand on that idea and **demonstrate how to define a function that will create and return an object of a given class**. The basic idea is that a **list is created with the relevant members, the list’s class is set, and a copy of the list is returned**.

### Here we examine two different ways to construct an S3 class. 
* The first approach is more commonly used and is more straightforward. It makes use of basic list properties. 
* The second approach makes use of the local environment within a function to define the variables tracked by the class. 

The advantage to the second approach is that **it looks more like the object oriented approach** that many are familiar with. The **disadvantage is that it is more difficult to read the code**, and it is more like working with pointers which is different from the way other objects work in R.

#### 3.1. Straight Forward Approach
The first approach is the more standard approach most often seen with S3 classes. It makes use of the methods defined outside of the class which is described below, Creating Methods. It also keeps track of the data maintained by the class using the standard practices associated with lists.

The basic idea is that a **function is defined which creates a list**. The data entries tracked by the class are defined in the list. 

* In the example below the defaults are specified in the argument list with default values assigned. A new class name is appended to the list’s classes, and the list is returned.
```
NorthAmerican <- function(eatsBreakfast=TRUE,myFavorite="cereal")
{

        me <- list(
                hasBreakfast = eatsBreakfast,
                favoriteBreakfast = myFavorite
       )

        ## Set the name for the class
        class(me) <- append(class(me),"NorthAmerican")
        return(me)
}
```
Once this definition is executed a new function is defined, called NorthAmerican. A new object of this class can be created by calling the function.

```
> bubba <- NorthAmerican()
> bubba
$hasBreakfast
[1] TRUE

$favoriteBreakfast
[1] "cereal"

attr(,"class")
[1] "list"          "NorthAmerican"
> bubba$hasBreakfast
[1] TRUE
>
> louise <- NorthAmerican(eatsBreakfast=TRUE,myFavorite="fried eggs")
> louise
$hasBreakfast
[1] TRUE

$favoriteBreakfast
[1] "fried eggs"

attr(,"class")
[1] "list"          "NorthAmerican"
```
### 3.2 Local Environment Approach

Another approach can be employed that makes use of the local environment within a function to access the variables. When we define methods with this approach later, Local Environment Approach, the results will look more like object oriented approaches seen in other languages.

The approach relies on the local scope created when a function is called. A new environment is created that can be identified using the environment command. The environment can be saved in the list created for the class, and the variables within this scope can then be accessed using the identification of the environment.

In the example below this approach appears to require more overhead. When we examine how to add external methods to the class the advantage will be a little clearer.
```
NordAmericain <- function(eatsBreakfast=TRUE,myFavorite="cereal")
{

      ## Get the environment for this
      ## instance of the function.
      thisEnv <- environment()

      hasBreakfast <- eatsBreakfast
      favoriteBreakfast <- myFavorite

      ## Create the list used to represent an
      ## object for this class
      me <- list(

        ## Define the environment where this list is defined so
        ## that I can refer to it later.
        thisEnv = thisEnv,

        ## The Methods for this class normally go here but are discussed
        ## below. A simple placeholder is here to give you a teaser....
        getEnv = function()
        {
              return(get("thisEnv",thisEnv))
        }

        )

      ## Define the value of the list within the current environment.
      assign('this',me,envir=thisEnv)

      ## Set the name for the class
      class(me) <- append(class(me),"NordAmericain")
      return(me)
}
Now that the class is defined, the environment used for a given object can be easily retrieved.

> bubba <- NordAmericain()
> get("hasBreakfast",bubba$getEnv())
[1] TRUE
> get("favoriteBreakfast",bubba$getEnv())
[1] "cereal"
Note that there is an unfortunate side effect to this approach. By keeping track of the environment, it is similar to using a pointer to the variables rather than the variables themselves. This means when you make a copy, you are making a copy of the pointer to the environment.

> bubba <- NordAmericain(myFavorite="oatmeal")
> get("favoriteBreakfast",bubba$getEnv())
[1] "oatmeal"
> louise <- bubba
> assign("favoriteBreakfast","toast",louise$getEnv())
> get("favoriteBreakfast",louise$getEnv())
[1] "toast"
> get("favoriteBreakfast",bubba$getEnv())
[1] "toast"
```

This issue will be explored again in the subsection below detailing how to create methods for an S3 class. If you wish to be able to copy an object using this approach you need to create a new method to return a proper copy.
