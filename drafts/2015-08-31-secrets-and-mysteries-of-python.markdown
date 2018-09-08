---
layout: post
title:  "Secrets and Mysteries of Python"
date:   2015-08-31 00:59:23
categories: python secrets mysteries
---

This post is entirely about my first love of life. Yes it is __Python__. I will
keep on adding more to this as I discover more and more about it.

* Function and Lists in Python

	I was randomly reading about Python on some blog, through I which I caem to know a
	wonderful thing about Python.
	Suppose you have a piece of code

	```
	def f(n,a=[]):
		a.append(n)
		return a

	print f(1)		# expected output : [1]
					# output produced : [1]
	print f(2)		# expected output : [2]
					# output produced : [1,2]
	```

	Now the question arises why this unusual behaviour. This I will cover when
	I learn more about it.

* List comprehension in Python

	These are the short ways to construct list in Python.
	Eg:
	 * A list of even numbers till 10

		 ```[x for x in range(11) if x%2==0]```

	 * A double for loop

		 ```
		 result = []
		 for tag in tags:
		 	for entry in entries:
		 		if tag in entry:
		 			result.extend(tag)
		 ```

		 The list comprehension for the above for loop will be

		 ```
		 [tag for tag in tags for entry in entries if tag in entry]
		 ```

		 This is because the order of loop inside list comprehension is based on
		 the order of loops in normal construct. Outer most loops comes first and
		 then inner loops comes subsequently.

		 If you have a single ```if``` condition then that goes at the end but if
		 you have a ```if-else``` condition, then that will be come at the
		 beginning like

		 ```
		 [tag if tag in entry else [] for tag in tags for entry in entries]
		 ```

	 * Suppose you have a list like

	 	```
	 	a = [[1,2],[3,4]]
	 	# you need something like [1,2,3,4]
	 	# list comprehension will be
	 	[x for b in a for x in b]
	 	```

	 * Now if you have two strings and you want all the possible combination
	 between those two strings, then

	 	```
	 	s1 = "abc"
	 	s2 = "def"
	 	[(x,y) for x in s1 for y in s2] 	#tuple wise notation
	 	# or
	 	[x+y for x in s1 for y in s2]
	 	```

 * Difference between ```extend``` and ```append```

 	```append``` inserts the element at the end of the list, whereas ```extend``` appends item from the iterable to the end of the array. Let us take some example to understand this

 	```
 	a = [1,2,3]
 	a.append([4,5])			# [1,2,3,[4,5]]
 	a.extend([4,5])			# [1,2,3,[4,5],4,5]
 	a.extend((6,7))			# [1,2,3,[4,5],4,5,6,7]
 	a.extend({"8":9})		# [1,2,3,[4,5],4,5,6,7,'8']
 	```

 * This one is really interesting. Sometimes we have a variable which contains a string. Now we want to use that string as a variable to assign. Lets make it clear with an example

 	```
 	trueCounter  = 0
 	falseCounter = 0
 	# now from somewhere lets say input i am getting a value which can be either trueCounter or falseCounter and I need to assign that a value.
 	# One way can be a if - else
 	name  = input()
 	count = input()
 	if count ==  "trueCounter":
 		trueCounter = count
 	elif count == "falseCounter":
 		falseCounter = count

 	# The other way to do this is an exec statement

 	exec("%s = %d" % (name,count)) 			# depending on the value of the name, it will assign
 											# the count

 	```
 * We all must have used ```split``` function at some point of time. But only few of us are aware of the fact that it takes a second optional argument. The second argument can be used as a check to see if the delimiter is present or not and then return the first n elements after splitting. The following example will make it more clear

 	```
 	a = 'b;c;d;e;f;g'
 	a.split(';')			# ['b', 'c', 'd', 'e', 'f', 'g']
 	a.split(';',1)			# ['b', 'c;d;e;f;g']
 	a.split(';',2)			# ['b', 'c', 'd;e;f;g']
 	a.split(';',6)			# ['b', 'c', 'd', 'e', 'f', 'g']
 	a.split(';',7)			# ['b', 'c', 'd', 'e', 'f', 'g']


 	```
 * `True` and `False` are keywords in Python 3 but not in Python 2. Hence this is possible in Python 2

 	`
 	True,False = False,True 				# not possible in Python 3
 	`




