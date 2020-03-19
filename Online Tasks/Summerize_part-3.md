lapply()
========

Looping on the command line
---------------------------

Writing for and while loops is useful when programming but not particularly easy when working interactively on the command line. There are some functions which implement looping to make life easier.

* lapply(): Loop over a list and evaluate and function on each element
* sapply(): Same as lapply() but try to simplify the result
* apply(): Apply a function over the margins of an array
* tapply(): Apply a function over subsets of a vector
* mapply(): Multivariate version of lapply

An auxiliary function split() is also useful, particularly in conjunction with lapply().

lapply()
--------

lapply() takes three arguments

1. A list x (if x is not a list it will be coerced to a list using as.list)
2. A function (or name of a function) FUN
3. Other arguments via its … argument

The function:

	lapply

	## function(X, FUN, …) {
	##     FUN <- match.fun(FUN)
	##     if (!is.vector(X) || is.object(X))
	##         X <- as.list(X)
	##     .Internal(lapply(X, FUN))
	## }
	## <bytecode: 0x7ff7a195c00>
	## <environment: namespace:base>

The actual looping is done internally in C code.

lapply() always returns a list, regardless of the class of the input.

	> x < list(a = 1:5, b = rnorm(10))
	> lapply(x, mean)
	$a
	[1] 3
	
	$b
	[1] 0.0296824

A more complicated example:

	> x <- list(a = 1:4, b = rnorm(10), c = rnorm(20, 1), d = rnorm(100, 5))
	> lapply(x, mean)
	$a
	[1] 2.5
	
	$b
	[1] 0.6082667
	
	$c
	[1] 1.467083
	
	$d
	[1] 5.074749

Another way to call lapply()

	> x <- 1:4
	> lapply(x, runif)
	[[1]]
	[1] 0.2675082
	
	[[2]]
	[1] 0.2186453 0.5167968
	
	[[3]]
	[1] 0.2689506 0.1811683 0.5185761
	
	[[4]]
	[1] 0.5627829 0.1291569 0.2563676 0.7179353

Pass arguments to your function:

	> x <- 1:4
	> lapply(x, runif, min = 0, max = 10)
	[[1]]
	[1] 2.7815

	[[2]]
	[1] 3.90933223 0.03094193

	[[3]]
	[1] 5.307777 8.944883 1.933448

	[[4]]
	[1] 2.6599025 0.5341006 8.7176168 3.2705339

lapply() and friends make heavy use of anonymous functions.

	> x <- list(a = matrix(1:4, 2, 2), b = matrix(1:6, 3, 2))
	> x
	$a
	     [,1] [,2]
	[1,]    1    3
	[2,]    2    4

	$b
	     [,1] [,2]
	[1,]    1    4
	[2,]    2    5
	[3,]    3    6

An anonymous function for extracting the first column of each matrix:

	> lapply(x, function(elt) elt[,1])
	$a
	[1] 1 2

	$b
	[1] 1 2 3

sapply()
--------

sapply() will try to simplify the result of lapply if possible

* If the result is a list where every element is length 1, then a vector is returned
* IF the result i a list where every element is a vector of the same length (> 1), a matrix is returned
* If it can't figure things out, a list is returned

Here's the lapply() behavior again:

	> x <- list(a = 1:4, b = rnorm(10), c = rnorm(20, 1), d = rnorm(100, 5))
	> lapply(x, mean)
	$a
	[1] 2.5

	$b
	[1] -0.21178

	$c
	[1] 1.073342

	$d
	[1] 5.050767

	> sapply(x, mean)
	        a         b         c         d 
 	2.500000 -0.211780  1.073342  5.050767
 
 apply()
=======

apply() is used to evaluate a function (often an anonymous one) over the margins of an array

* It is most often used to apply a function to the rows or columns of a matrix
* It can be used with general array _e.g._ taking the average of an array of matrices
* It is not only faster than writing a loop, but it works in one line!

Example:

	> str(apply)
	function (X, MARGIN, …)

Arguments:

* X is an array
* MARGIN is an integer vector indicating which margins should be "retained"
* FUN is a function to be applied
* … is for other arguments to be passed to FUN

Example:

	> x <- matrix(rnorm(200), 20, 10)
	> apply(x, 2, mean)
	 [1]  -0.24271338 -0.09769539 -0.26258277
     [4]  -0.38282262  0.36272239 -0.23001762
	 [7]  -0.30442240  0.18822820  0.23424057
	 [10] -0.35909576
	
	> apply(x, 1, sum)
	 [1]  -0.409985811 -3.268890256 -3.728752232
	 [4]   0.436631813 -0.455544538 -6.780291602
     [7]  -2.008526814  2.271598286 -0.001329909
	 [10] -0.772003961  2.252825741  0.326282318
     [13] -4.396864141 -3.975331909 -3.523458630
     [16]  1.877565192  2.122820628 -4.235880427
     [19] -1.915425553  4.301386289


Col/row sums and means
----------------------

For sums and means of matrix dimensions, we have some shortcuts.

* rowSums = apply(x, 1, sum)
* rowMeans = apply(x, 1, mean)
* colMeans = apply(x, 2, sum)
* colMeans = apply(x, 2, mean)

The shortcut functions are _much_ faster, but you won't notice unless you're using a large matrix.


Other ways to apply
-------------------

Quantiles of the rows of a matrix.

	> x <- matrix(rnorm(200), 20, 10)
	> apply(x, 1, quantile, probs = c(0.25, 0.75))
	            [,1]       [,2]       [,3]       [,4]
	25% -1.463023845 -0.2932014 -0.4127894 -0.8502298
	75% -0.006850471  0.7238166  0.7484396  0.3859197
	          [,5]       [,6]       [,7]       [,8]
	25% -0.2544152 -0.5906017 -0.1367046 -0.8027295
	75%  0.6316449  0.3365734  0.4772425  0.6892627
	          [,9]      [,10]      [,11]     [,12]
	25% -0.3477938 -0.7819127 -1.5651389 -1.300000
	75%  1.2975913  0.6874982  0.2340079  1.030404
	          [,13]      [,14]      [,15]      [,16]
	25% -0.09854616 -0.6715086 -0.9392777 -0.2083622
	75%  1.02289904  0.4008069  0.7483483  0.8404269
	         [,17]      [,18]      [,19]      [,20]
	25% -0.6191729 -0.7834631 -1.1424522 -0.3559212
	75%  0.9909588  0.8844815  0.7276243  0.7105771

Average matrix in an array

	> a <- array(rnorm(2 * 2 * 10), c(2, 2, 10))
	> apply(a, c(1, 2), mean)
	            [,1]       [,2]
	[1,]  0.04107431 0.64450175
	[2,] -0.04269955 0.07289623
	> rowMeans(a, dims = 2)
	            [,1]       [,2]
	[1,]  0.04107431 0.64450175
	[2,] -0.04269955 0.07289623
  
  tapply()
--------

tapply() is used to apply a function over subsets of a vector. I don't know why it's called tapply().

	> str(tapply)
	function (X, INDEX, FUN = NULL, ..., simplify = TRUE)

Arguments:

* X is a vector
* INDEX is a factor or list of factors (or else they are coerced to factors)
* FUN is a function to be applied
* … contains other arguments to be passed to FUN
* simplify, should we simplify the result?

Take group means.

	> x <- c(rnorm(10), runif(10), rnorm(10, 1))
	> f <- gl(3, 10)
	> f
	 [1] 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3 3 3 3 3 3 3 3 3
	Levels: 1 2 3
	> tapply(x, f, mean)
	        1         2         3 
	0.4102277 0.6013687 0.7498127

Take group means without simplification.

	> tapply(x, f, mean, simplify = FALSE)
	$`1`
	[1] 0.4102277
	
	$`2`
	[1] 0.6013687
	
	$`3`
	[1] 0.7498127

Find group ranges.

	> tapply(x, f, range)
	$`1`
	[1] -2.378255  2.673336

	$`2`
	[1] 0.1150650 0.9278384

	$`3`
	[1] -0.2201542  1.5680852

tapply()
--------

tapply() is used to apply a function over subsets of a vector. I don't know why it's called tapply().

	> str(tapply)
	function (X, INDEX, FUN = NULL, ..., simplify = TRUE)

Arguments:

* X is a vector
* INDEX is a factor or list of factors (or else they are coerced to factors)
* FUN is a function to be applied
* … contains other arguments to be passed to FUN
* simplify, should we simplify the result?

Take group means.

	> x <- c(rnorm(10), runif(10), rnorm(10, 1))
	> f <- gl(3, 10)
	> f
	 [1] 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3 3 3 3 3 3 3 3 3
	Levels: 1 2 3
	> tapply(x, f, mean)
	        1         2         3 
	0.4102277 0.6013687 0.7498127

Take group means without simplification.

	> tapply(x, f, mean, simplify = FALSE)
	$`1`
	[1] 0.4102277
	
	$`2`
	[1] 0.6013687
	
	$`3`
	[1] 0.7498127

Find group ranges.

	> tapply(x, f, range)
	$`1`
	[1] -2.378255  2.673336

	$`2`
	[1] 0.1150650 0.9278384

	$`3`
	[1] -0.2201542  1.5680852

tapply()
--------

tapply() is used to apply a function over subsets of a vector. I don't know why it's called tapply().

	> str(tapply)
	function (X, INDEX, FUN = NULL, ..., simplify = TRUE)

Arguments:

* X is a vector
* INDEX is a factor or list of factors (or else they are coerced to factors)
* FUN is a function to be applied
* … contains other arguments to be passed to FUN
* simplify, should we simplify the result?

Take group means.

	> x <- c(rnorm(10), runif(10), rnorm(10, 1))
	> f <- gl(3, 10)
	> f
	 [1] 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3 3 3 3 3 3 3 3 3
	Levels: 1 2 3
	> tapply(x, f, mean)
	        1         2         3 
	0.4102277 0.6013687 0.7498127

Take group means without simplification.

	> tapply(x, f, mean, simplify = FALSE)
	$`1`
	[1] 0.4102277
	
	$`2`
	[1] 0.6013687
	
	$`3`
	[1] 0.7498127

Find group ranges.

	> tapply(x, f, range)
	$`1`
	[1] -2.378255  2.673336

	$`2`
	[1] 0.1150650 0.9278384

	$`3`
	[1] -0.2201542  1.5680852

split()
=======

split takes a vector or other objects and splits it into groups determined by a factor or a list of factors.

	> str(split)
	function (x, f, drop = FALSE, …)

Arguments:

* x is a vector (or list) or data frame
* f is a factor (or coerced to one) or a list of factors
* drop indicates whether empty factor levels should be dropped

Example:

	> x <- c(rnorm(10), runif(10), rnorm(10, 1))
	> f <- gl(3, 10)
	> split(x, f)
	$`1`
	 [1] -0.5615980 -1.0227400  2.3743946 -1.1828731
	 [5] -0.4588891  1.3406054 -1.2529312 -1.5232924
	 [9] -0.9483891  0.5974399

	$`2`
	 [1] 0.28819663 0.24568527 0.08535357 0.64725198
	 [5] 0.55680796 0.99364603 0.25011440 0.90709685
	 [9] 0.20941979 0.63402239

	$`3`
	 [1]  0.6468613 -0.3783117 -0.8270978  0.3052268
	 [5]  0.9999587  1.0533331  2.4046919  1.3490653
	 [9]  0.6020979  1.3048118

A common idiom is split() followed by an lapply()

	> lapply(split(x, f), mean)
	$`1`
	[1] -0.2638273

	$`2`
	[1] 0.4817595

	$`3`
	[1] 0.7460637


Splitting a Data Frame
----------------------

	> library(datasets)
	> head(airquality)
	  Ozone Solar.R Wind Temp Month Day
	1    41     190  7.4   67     5   1
	2    36     118  8.0   72     5   2
	3    12     149 12.6   74     5   3
	4    18     313 11.5   62     5   4
	5    NA      NA 14.3   56     5   5
	6    28      NA 14.9   66     5   6

	> s <- split(airquality, airquality$Month)
	> lapply(s, function(x) colMeans(x[, c("Ozone", "Solar.R", "Wind")]))
	$`5`
	   Ozone  Solar.R     Wind 
	      NA       NA 11.62258 

	$`6`
	    Ozone   Solar.R      Wind 
	       NA 190.16667  10.26667 

	$`7`
	     Ozone    Solar.R       Wind 
	        NA 216.483871   8.941935 

	$`8`
	   Ozone  Solar.R     Wind 
	      NA       NA 8.793548 

	$`9`
	   Ozone  Solar.R     Wind 
	      NA 167.4333  10.1800

You can also use sapply() here.

	> sapply(s, function(x) colMeans(x[, c("Ozone", "Solar.R", "Wind")]))
	               5         6          7        8        9
	Ozone         NA        NA         NA       NA       NA
	Solar.R       NA 190.16667 216.483871       NA 167.4333
	Wind    11.62258  10.26667   8.941935 8.793548  10.1800

Remove NAs

	> sapply(s, function(x) colMeans(x[, c("Ozone", "Solar.R", "Wind")], na.rm  = TRUE))
	                5         6          7          8         9
	Ozone    23.61538  29.44444  59.115385  59.961538  31.44828
	Solar.R 181.29630 190.16667 216.483871 171.857143 167.43333
	Wind     11.62258  10.26667   8.941935   8.793548  10.18000


Splitting on More than One Level
--------------------------------

	> x <- rnorm(10)
	> f1 <- gl(2, 5)
	> f2 <- gl(5, 2)
	> f1
	 [1] 1 1 1 1 1 2 2 2 2 2
	Levels: 1 2
	> f2
	 [1] 1 1 2 2 3 3 4 4 5 5
	Levels: 1 2 3 4 5
	> interaction(f1, f2)
	 [1] 1.1 1.1 1.2 1.2 1.3 2.3 2.4 2.4 2.5 2.5
	Levels: 1.1 2.1 1.2 2.2 1.3 2.3 1.4 2.4 1.5 2.5

Interactions can create empty levels.

	> str(split(x, list(f1, f2)))
	List of 10
	 $ 1.1: num [1:2] 1.829 -0.808
	 $ 2.1: num(0) 
	 $ 1.2: num [1:2] -1.329 0.544
	 $ 2.2: num(0) 
	 $ 1.3: num 0.477
	 $ 2.3: num -0.108
	 $ 1.4: num(0) 
	 $ 2.4: num [1:2] -0.44 0.394
	 $ 1.5: num(0) 
	 $ 2.5: num [1:2] 1.333 -0.414

Use drop = TRUE to get rid of empty levels.

	> str(split(x, list(f1, f2), drop = TRUE))
	List of 6
	 $ 1.1: num [1:2] 1.829 -0.808
	 $ 1.2: num [1:2] -1.329 0.544
	 $ 1.3: num 0.477
	 $ 2.3: num -0.108
	 $ 2.4: num [1:2] -0.44 0.394
	 $ 2.5: num [1:2] 1.333 -0.414
   
   
   mapply()
========

mapply() is a multivariate apply of sorts which applies a function in parallel over a set of arguments.

	> str(mapply)
	function (FUN, ..., MoreArgs = NULL, SIMPLIFY = TRUE,
	          USE.NAMES = TRUE)

Arguments:

* FUN is a function to apply
* … contains arguments to apply over
* MoreArgs is a list of other arguments to FUN
* SIMPLIFY indicates whether the result should be simplified

The following is tedious to type:

	list(rep(1, 4), rep(2, 3), rep(3, 2), rep(4, 1))

Instead we can do:

	> mapply(rep, 1:4, 4:1)
	[[1]]
	[1] 1 1 1 1

	[[2]]
	[1] 2 2 2

	[[3]]
	[1] 3 3

	[[4]]
	[1] 4


Vectorizing a function
----------------------

Define a custom function:

	> noise <- function(n, mean, sd) {
	+ rnorm(n, mean, sd)
	+ }
	> noise(5, 1, 2)
	[1] 1.9218170  1.3781843 -0.5607748  3.7576328
	[5] 0.2590522
	> noise(1:5, 1:5, 2)
	[1] 1.780955 1.813852 1.827005 1.144564
	[5] 1.842238

Passing lists to this function does not behave as expected.

	> mapply(noise, 1:5, 1:5, 2)
	[[1]]
	[1] 2.2496

	[[2]]
	[1] 3.003979 3.621779

	[[3]]
	[1] 2.304840 4.135399 3.633926

	[[4]]
	[1] 1.269779 4.212011 2.963119 4.499524

	[[5]]
	[1] 5.353751 8.353743 6.389625 1.016071 3.344346

Which is the same as:

	> list(noise(1, 1, 2), noise(2, 2, 2),
	       noise(3, 3, 2), noise(4, 4, 2),
	       noise(5, 5, 2))
         
Debugging - Part 1
==================

Indications that something is not right.

* message
	* A generic notification/diagnostic message produced by the message function
	* Execution of the function continues
* warning
	* An indication that something is wrong but not necessarily fatal
	* Execution of the function continues
	* Generation by the warning function
* error
	* An indication that a fatal problem has occurred
	* Execution stops
	* Produced by the stop function
* condition
	* A generic concept for indicating that something unexpected can occur
	* Programmers can create their own conditions

Warnings:

	> log(-1)
	[1] NaN
	Warning message:
	In log(-1) : NaNs produced

Example:

	> printmessage <- function(x) {
	+ 	if(x > 0)
	+ 		print("x is greater than zero")
	+ 	else
	+ 		print("x is less than or equal to zero")
	+ 	invisible(x)
	+ }

	> printmessage(1)
	[1] "x is greater than zero"

	> printmessage(NA)
	Error in if (x > 0) print("x is greater than zero") else print("x is less than or equal to zero") : missing value where TRUE/FALSE needed

Another example:

	> printmessage2 <- function(x) {
	+ 	if(is.na(x))
	+ 		print("x is a missing value!")
	+ 	else if(x > 0)
	+ 		print("x is greater than zero")
	+ 	else
	+ 		print("x is less than or equal to zero")
	+ 	invisible(x)
	+ }
	> x <- log(-1)
	Warning message:
	In log(-1) : NaNs produced
	> printmessage2(x)
	[1] "x is a missing value!"

How do you know that something is wrong with your function?

* What was you input? How did you call the function?
* What were you expecting? Output, message, other results?
* What did you get?
* How does what you get differ from what you were expecting?
* Were your expectations correct in the first place?
* Can you reproduce the problem (exactly)?

