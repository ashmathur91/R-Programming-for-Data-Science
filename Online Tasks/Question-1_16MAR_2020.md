# S3 Class/Methods for Polygons

## The S3 system doesn’t have a formal way to define a class but typically, we use a list to define the class and elements of the list serve as data elements.

Here is our definition of a polygon represented using Cartesian coordinates. The class contains an element called xcoord and ycoord for the x- and y-coordinates, respectively. The make_poly() function is the “constructor” function for polygon objects. It takes as arguments a numeric vector of x-coordinates and a corresponding numeric vector of y-coordinates.

## Constructor function for polygon objects
## x a numeric vector of x coordinates
## y a numeric vector of y coordinates

```
make_poly <- function(x, y) {
        if(length(x) != length(y))
                stop("'x' and 'y' should be the same length")
        
        ## Create the "polygon" object 
        object <- list(xcoord = x, ycoord = y)
        
        ## Set the class name
        class(object) <- "polygon"
        object
}
```
Now that we have a class definition, we can develop some methods for operating on objects from that class.

The first method we’ll define is the print() method. The print() method should just show some simple information about the object and should not be too verbose—just enough information that the user knows what the object is.

Here the print() method just shows the user how many vertices the polygon has. It is a convention for print() methods to return the object x invisibly.

## Print method for polygon objects
## x an object of class "polygon"
```
print.polygon <- function(x, ...) {
        cat("a polygon with", length(x$xcoord), 
            "vertices\n")
        invisible(x)
}
```
Next is the summary() method. The summary() method typically shows a bit more information and may even do some calculations. This summary() method computes the ranges of the x- and y-coordinates.

The typical approach for summary() methods is to allow the summary method to compute something, but to not print something. The strategy is

The summary() method returns an object of class “summary_‘class name’”

There is a separate print() method for “summary_‘class name’” objects.

For example, here is the summary() method.

## Summary method for polygon objects
## object an object of class "polygon"
```
summary.polygon <- function(object, ...) {
        object <- list(rng.x = range(object$xcoord),
                       rng.y = range(object$ycoord))
        class(object) <- "summary_polygon"
        object
}
```
Note that it simply returns an object of class summary_polygon. Now the corresponding print() method.

## Print method for summary.polygon objects
## x an object of class "summary_polygon"

```
print.summary_polygon <- function(x, ...) {
        cat("x:", x$rng.x[1], "-->", x$rng.x[2], "\n")
        cat("y:", x$rng.y[1], "-->", x$rng.y[2], "\n")
        invisible(x)
}

```
Now we can make use of our new class and methods.

## Construct a new "polygon" object
```
x <- make_poly(1:4, c(1, 5, 2, 1))
```
We can use the print() to see what the object is.

print(x)
a polygon with 4 vertices
And we can use the summary() method to get a bit more information about the object.
```
out <- summary(x)
class(out)
[1] "summary_polygon"
print(out)
x: 1 --> 4 
y: 1 --> 5 
Because of auto-printing we can just call the summary() method and let the results auto-print.
```
```
summary(x)
x: 1 --> 4 
y: 1 --> 5 
From here, we could build other methods for interacting with our polygon object. For example, it may make sense to define a plot() method or maybe methods for intersecting two polygons together.
```
