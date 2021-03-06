# BackTrackLib

My own take on backtracking algorithms (w or w/o recursion). They are generalizations of the typical processes so as to permit any posible implementation while using whatever data-structures and heuristic functions.

## Recursive Backtracking with Solver class

### Overview:

` Solver(<calculate_posibles_func>, <basecase_func>) `

Solver class is designed to compute and represent answers to all backtracking problems. It receives two user defined functions at initialization:

`calculate_posibles_func` receives a list with earlier choices and must return an iterable with posible choices for the current step.

`basecase_func` receives a list with earlier choices and returns a boolean value to indicate it has reached a valid answer or not.

### Using `Solver` to find answers:

Answers are not computed when the object is created. If no solutions have been computed yet, upon accesing `.solutions` of a object of class `Solver` one solution will be computed and returned automatically. Alternatively the `.solve` method can be used.

#### Atributes:
` .solutions ` is a list with all answers found, `.time` is the time it took (in seconds) to compute, and ` found_all` is a boolean value to indicate whether it found all the answers requested or not. This value will always be True if all possible values were found even if the quantity is lower than `number_of_answers`, so ` found_all` will only be false if the time limit was reached.

More granular control over the amount of solutions or the maximum time the search should take can be had by using the `.solve` method.

`<solver_object>.solve([number_of_answers=1], [max_time=0], [threading=False])`

`number_of_answers` is an integer which can be passed to indicate the number of answers to be calculated. Default value is 1. If number of answers is set to 0 then all posible answers will be computed.

`max_time ` is an integer which can be passed to indicate the maximum time the algorithm should run (in seconds). Default value (0) permits it to run undisturbed. Should the time limit be reached, the algorithm will return all answers found up to this point.

`threading ` is a boolean value that indicates whether threads should be implemented in the first step or not. The threads will cease function once the number of answers computed is equal to the number asked by the user, but if two threads reach diferent answers at roughly the same time more answers than requested can be returned.

#### Example code: 8 queens

~~~python
from backtracklib import Solver

# 8 queens problem: position 8 queens on a chessboard so that no one attacks another.

def basecase(parcial):
	if len(parcial) == 8:
		return True
	return False

def calculate_posibles(parcial):
	ret = []
	for x in range(8):
		for y in range(8):
			is_in = False
			for i in range(8):
				if (x,i) in parcial or (i, y) in parcial or (x-i, y-i) in parcial or (x+i, y+i) in parcial:
					is_in = True
			if not is_in: 
				ret.append((x,y))
	return ret

gen = Solver(calculate_posibles, basecase)
answer1 = gen.solutions[0]		# implicit solving
~~~

#### Example code: Knapsack Problem

![MacDown logo](https://imgs.xkcd.com/comics/np_complete.png)

Source: [xkcd](https://xkcd.com/287/)

~~~python
from backtracklib import Solver

def total(parcial):
	total = 0
	for elem in parcial:
		total += elem.price
	return total

def basecase(parcial):
	if total(parcial) == "1505":
		return True
	return False

def calculate_posibles(parcial):
	posibles = []
	for elem in apetizers:	# apetizers should be a list of objects that have a price atribute
		if total(parcial) + elem.price <= 1505:
			posibles.append(elem)
	return posibles
			

gen = Solver(calculate_posibles, basecase)
answers = gen.solve(num_answers=4, threading=True)	# explicit solving

~~~

### Solution Trees:

Solver class objects also include a `.tree` atribute which represents the recursion tree. This atribute can be printed using the python built-in method `print(<solver_object>.tree)`.

#### Example output of 4 queens (simplified version of 8 queens)

~~~
(0, 0)>	(1, 2)>	(2, 1)
		(3, 1)
	(1, 3)>	(2, 1)
		(3, 1)
		(3, 2)
	(2, 1)>	(1, 2)
		(1, 3)
	(2, 3)>	(3, 1)
		(3, 2)
	(3, 1)>	(1, 2)
		(1, 3)
		(2, 3)
	(3, 2)>	(1, 3)
		(2, 3)
(0, 1)>	(1, 0)>	(2, 2)
		(3, 3)
	(1, 3)>	(2, 0)>	(3, 2)
~~~


### Optimizations and heuristics:

All user entered heuristics and optimizations should be implemented in ` calculate_posibles_func `. A general rule of thumb to having a somewhat optimized algorithm is to build ` calculate_posibles_func ` in such a way that `basecase` has to check as little conditions as possible. 

In some problems inputs can be subdivided into discrete sets of elements such that no posible answer will ever have two elements of the same set (ie. in the above example no two queens will ever be in the same column). In these cases its generally more efficient to only have ` calculate_posibles_func ` return valid elements of one set instead of all valid elements.

If a discretization such as described in the paragraph above is possible then the algorithm can be further optimized by ordering the sets in such a way that ` calculate_posibles_func ` will access first the sets with the most elements.

Heuristics are dificult to generalize (normally unique to the problem). Some more general ones will maybe be one day implemented but for now they must be defined by the user and implemented in ` calculate_posibles_func `. Heuristics that asign a higher probability of succes to any element within the set of posibles should be implemented by sorting the return iterable of ` calculate_posibles_func ` by probability (in ascending order->from lower to higher probability).

## NonRecursive Backtracking with NonSolver class
In development stage, this is another generalized take on backtracking implemented w/o recursive function calls so as to avoid stack calls and recursion errors.

### Overview:
` NonSolver(<calculate_posibles_func>, <basecase_func>) `
The `NonSolver` class receives the same arguments as `Solver`, and only diferentiates itself in that its ` .solutions ` atribute is a generator class object that computes answers when requested.

All the same comments on the implementation of heuristics apply.

#### Example code: non recursive solution to 8 queens

~~~python
from non_recursive_backtracking import NonSolver

# 8 queens problem: position 8 queens on a chessboard so that no one attacks another.

def basecase(parcial):
	if len(parcial) == 8:
		return True
	return False

def calculate_posibles(parcial):
	ret = []
	for x in range(8):
		for y in range(8):
			is_in = False
			for i in range(8):
				if (x,i) in parcial or (i, y) in parcial or (x-i, y-i) in parcial or (x+i, y+i) in parcial:
					is_in = True
			if not is_in: 
				ret.append((x,y))
	return ret

# uses a non recursive algorithm to backtrack

queen_solver = NonSolver(calculate_posibles, basecase)
print(next(queen_solver.solutions))
~~~
