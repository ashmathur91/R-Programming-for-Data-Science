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

********

### 4. Creating Methods

Now explore **how to create methods associated with a class**. Again we break it up into two parts. 
* The first approach is used for both approaches discussed in the previous section.

#### 4.1. Straight Forward Approach
The first approach is to define a function that exists outside of the class. The function is defined in a generic way, and then a function specific to a given class is defined. The R environment then decides which function to use based on the class names of an argument to the function, and the suffix used in the names of the associated functions.

One thing to keep in mind is that for ** assignment R makes copies of objects**. The implication is that if you change a part of an object you need to return an exact copy of the object. Otherwise your changes may be lost.

In the examples below we define accessors for the variables in the class defined above. We assume that the class NorthAmerican is defined in the same way as the first example above, Straightforward Class. In the first example the goal is that we want to create a function that will set the value of hasBreakfast for a given object. The name of the function will be setHasBreakfast.

The first step is to reserve the name of the function, and use the UseMethod command to tell R to search for the correct function. If we pass an object whose class includes the name NorthAmerican then the correct function to call should be called setHasBreakfast.NorthAmerican. Note that we will also create a function called setHasBreakfast.default. This function will be called if R cannot find another function of the correct class.

```
setHasBreakfast <- function(elObjeto, newValue)
        {
                print("Calling the base setHasBreakfast function")
                UseMethod("setHasBreakfast",elObjeto)
                print("Note this is not executed!")
        }

setHasBreakfast.default <- function(elObjeto, newValue)
        {
                print("You screwed up. I do not know how to handle this object.")
                return(elObjeto)
        }


setHasBreakfast.NorthAmerican <- function(elObjeto, newValue)
        {
                print("In setHasBreakfast.NorthAmerican and setting the value")
                elObjeto$hasBreakfast <- newValue
                return(elObjeto)
        }
```
The first thing to note is that the function returns a copy of the object passed to it. R passes copies of objects to functions. If you change an object within a function it does not change the original object. You must pass back a copy of the updated object.
```
> bubba <- NorthAmerican()
> bubba$hasBreakfast
[1] TRUE
> bubba <- setHasBreakfast(bubba,FALSE)
[1] "Calling the base setHasBreakfast function"
[1] "In setHasBreakfast.NorthAmerican and setting the value"
> bubba$hasBreakfast
[1] FALSE
> bubba <- setHasBreakfast(bubba,"No type checking sucker!")
[1] "Calling the base setHasBreakfast function"
[1] "In setHasBreakfast.NorthAmerican and setting the value"
> bubba$hasBreakfast
[1] "No type checking sucker!"
If the correct function cannot be found then the default version of the function is called.

> someNumbers <- 1:4
> someNumbers
[1] 1 2 3 4
> someNumbers <- setHasBreakfast(someNumbers,"what?")
[1] "Calling the base setHasBreakfast function"
[1] "You screwed up. I do not know how to handle this object."
> someNumbers
[1] 1 2 3 4
It is a good practice to only use predefined accessors to get and set values held by an object. As a matter of completeness we define methods to get the value of the hasBreakfast field.

getHasBreakfast <- function(elObjeto)
        {
                print("Calling the base getHasBreakfast function")
                UseMethod("getHasBreakfast",elObjeto)
                print("Note this is not executed!")
        }

getHasBreakfast.default <- function(elObjeto)
        {
                print("You screwed up. I do not know how to handle this object.")
                return(NULL)
        }


getHasBreakfast.NorthAmerican <- function(elObjeto)
        {
                print("In getHasBreakfast.NorthAmerican and returning the value")
                return(elObjeto$hasBreakfast)
        }
The functions to get the values are used in the same way.

> bubba <- NorthAmerican()
> bubba <- setHasBreakfast(bubba,"No type checking sucker!")
[1] "Calling the base setHasBreakfast function"
[1] "In setHasBreakfast.NorthAmerican and setting the value"
> result <- getHasBreakfast(bubba)
[1] "Calling the base getHasBreakfast function"
[1] "In getHasBreakfast.NorthAmerican and returning the value"
> result
```
[1] "No type checking !"

#### 4.2. Local Environment Approach
If the second method for defining an S3 class is used as seen above, Local Environment Class, then the approach for defining a method can include an additional way to define a method. In this approach functions can be defined within the list that defines the object.
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

              ## Define the accessors for the data fields.
              getEnv = function()
              {
                      return(get("thisEnv",thisEnv))
              },

              getHasBreakfast = function()
              {
                      return(get("hasBreakfast",thisEnv))
              },

              setHasBreakfast = function(value)
              {
                      return(assign("hasBreakfast",value,thisEnv))
              },


              getFavoriteBreakfast = function()
              {
                      return(get("favoriteBreakfast",thisEnv))
              },

              setFavoriteBreakfast = function(value)
              {
                      return(assign("favoriteBreakfast",value,thisEnv))
              }

        )

      ## Define the value of the list within the current environment.
      assign('this',me,envir=thisEnv)

      ## Set the name for the class
      class(me) <- append(class(me),"NordAmericain")
      return(me)
}
With this definition the methods can be called in a more direct manner.

> bubba <- NordAmericain(myFavorite="oatmeal")
> bubba$getFavoriteBreakfast()
[1] "oatmeal"
> bubba$setFavoriteBreakfast("plain toast")
> bubba$getFavoriteBreakfast()
[1] "plain toast"
```
As noted above, Local Environment Class, this approach can be problematic when making a copy of an object. If you need to make copies of your objects a function must be defined to explicitly make a copy.
```
makeCopy <- function(elObjeto)
        {
                print("Calling the base makeCopy function")
                UseMethod("makeCopy",elObjeto)
                print("Note this is not executed!")
        }

makeCopy.default <- function(elObjeto)
        {
                print("You screwed up. I do not know how to handle this object.")
                return(elObjeto)
        }


makeCopy.NordAmericain <- function(elObjeto)
        {
                print("In makeCopy.NordAmericain and making a copy")
                newObject <- NordAmericain(
                        eatsBreakfast=elObjeto$getHasBreakfast(),
                        myFavorite=elObjeto$getFavoriteBreakfast())
                return(newObject)
        }
With this definition we can now make a proper copy of the object and get the expected results.

> bubba <- NordAmericain(eatsBreakfast=FALSE,myFavorite="oatmeal")
> louise <- makeCopy(bubba)
[1] "Calling the base makeCopy function"
[1] "In makeCopy.NordAmericain and making a copy"
> louise$getFavoriteBreakfast()
[1] "oatmeal"
> louise$setFavoriteBreakfast("eggs")
> louise$getFavoriteBreakfast()
[1] "eggs"
> bubba$getFavoriteBreakfast()
[1] "oatmeal"
```
## 5. Inheritance
Inheritance is part of what makes it worthwhile to go to the effort of making up a proper class. The basic idea is that another class can be constructed that makes use of all the data and methods of a base class and builds on them by adding additional data and methods.

The basic idea is that an object’s class is a vector that contains an ordered list of classes that an object is a member of. When a new object is created it can add its class name to that list. The methods associated with the class can use the NextMethod command to search for the function associated with the next class in the list.

In the examples below we build on the example of the NorthAmerican class defined above. The examples below assume that the NorthAmerican class given above is defined, and the class hierarchy is shown in Figure 1..

Diagram of the NorthAmerican derived classes.
Figure 1.

Diagram of the NorthAmerican derived classes.
A file with the full script is available at s3Inheritance.R . We do not provide the full code set here in order to keep the discussion somewhat under control. Our first step is to define the new classes.
```
Mexican <- function(eatsBreakfast=TRUE,myFavorite="los huevos")
{

        me <- NorthAmerican(eatsBreakfast,myFavorite)

        ## Add the name for the class
        class(me) <- append(class(me),"Mexican")
        return(me)
}


USAsian <- function(eatsBreakfast=TRUE,myFavorite="pork belly")
{

        me <- NorthAmerican(eatsBreakfast,myFavorite)

        ## Add the name for the class
        class(me) <- append(class(me),"USAsian")
        return(me)
}

Canadian <- function(eatsBreakfast=TRUE,myFavorite="back bacon")
{

        me <- NorthAmerican(eatsBreakfast,myFavorite)

        ## Add the name for the class
        class(me) <- append(class(me),"Canadian")
        return(me)
}

Anglophone <- function(eatsBreakfast=TRUE,myFavorite="pancakes")
{

        me <- Canadian(eatsBreakfast,myFavorite)

        ## Add the name for the class
        class(me) <- append(class(me),"Anglophone")
        return(me)
}

Francophone <- function(eatsBreakfast=TRUE,myFavorite="crepes")
{

        me <- Canadian(eatsBreakfast,myFavorite)

        ## Add the name for the class
        class(me) <- append(class(me),"Francophone")
        return(me)
}
```
With these definitions we can define an object. In this case we create an object from the Francophone class.
```
> francois <- Francophone()
> francois
$hasBreakfast
[1] TRUE

$favoriteBreakfast
[1] "crepes"

attr(,"class")
[1] "list"          "NorthAmerican" "Canadian"      "Francophone"
```
The thing to notice is that the class vector demonstrates the class inheritance structure. We can now define a method, makeBreakfast which will make use of the class structure.
```
makeBreakfast <- function(theObject)
        {
                print("Calling the base makeBreakfast function")
                UseMethod("makeBreakfast",theObject)
        }

makeBreakfast.default <- function(theObject)
        {
                print(noquote(paste("Well, this is awkward. Just make",
                                  getFavoriteBreakfast(theObject))))
                return(theObject)
        }

makeBreakfast.Mexican <- function(theObject)
        {
                print(noquote(paste("Estoy cocinando",
                                       getFavoriteBreakfast(theObject))))
                NextMethod("makeBreakfast",theObject)
                return(theObject)
        }

makeBreakfast.USAsian <- function(theObject)
        {
                print(noquote(paste("Leave me alone I am making",
                                               getFavoriteBreakfast(theObject))))
                NextMethod("makeBreakfast",theObject)
                return(theObject)
        }

makeBreakfast.Canadian <- function(theObject)
        {
                print(noquote(paste("Good morning, how would you like",
                                       getFavoriteBreakfast(theObject))))
                NextMethod("makeBreakfast",theObject)
                return(theObject)
        }

makeBreakfast.Anglophone <- function(theObject)
        {
                print(noquote(paste("I hope it is okay that I am making",
                                       getFavoriteBreakfast(theObject))))
                NextMethod("makeBreakfast",theObject)
                return(theObject)
        }

makeBreakfast.Francophone <- function(theObject)
        {
                print(noquote(paste("Je cuisine",
                                       getFavoriteBreakfast(theObject))))
                NextMethod("makeBreakfast",theObject)
                return(theObject)
        }
```
**Note** that the functions call the NextMethod function to call the next function in the list of classes.
```
> francois <- makeBreakfast(francois)
[1] "Calling the base makeBreakfast function"
[1] Good morning, how would you like crepes
[1] Je cuisine crepes
[1] Well, this is awkward. Just make crepes
```
