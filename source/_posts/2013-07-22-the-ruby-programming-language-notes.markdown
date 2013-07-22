---
layout: post
title: "The Ruby Programming Language Notes"
date: 2013-07-22 11:14
comments: true
categories: 
---

- Global variables are prefixed with $
- Instance variables are prefixed with @
- Class variables are prefixed with @@
- Constant begin with a capital letter

```
/[Rr]uby/   # Matches "Ruby" or "ruby"
/\d{5}/     # Matches 5 consecutive digits
1..3        # All x where 1 <= x <= 3 # has equal both side
1...3       # All x where 1 <= x < 3
```

- Ruby case statement matches its expression against each of the possible cases using `===`

- Ruby's strings are mutable, call freeze method to prevent any future modifications

- In Ruby world, only nil and false are false, anything else are true

- `print` is almost same with `puts`, except it does not append a newline

- `p` almost same with `puts`, except it use `inspect` rather than `to_s`

- lexical
- punctuation
- syntactic
- arithmetic

- All numeric objects are immutable

- Integer division by zero causes a ZeroDivisionError to be thrown.
- Floating-point division by zero does not cause an error; it simple returns the value Infinity
- 0.0/0.0 => NaN

- `4**3**2` is same as `4**9` not `64**2`

- even = (x[0] == 0)  # A number is even if the least-significat bit is 0

- 'a\b' == 'a\\b'

- money = "\u{20AC 20 A3 20 A5}" # => "€ £ ¥"

- Run system command
```
`ls`
%x[ls]
```

?A  # Character literal for the  ASCII character A
/"  # Character literal for the  ASCII character "
/?  # Character literal for the  ASCII character ?
# not need any more

```
a = 0;
"#{a=a+1} " * 3   # returns "1 1 1", not "1 2 3"
```

```
s = "hello"
s[index, length]
s[0, 2] # "he"
s[start_index..end_index]   # include end_index
s[start_index...end_index]  # not include end_index
s[/[aeiou]/] = '*'          # replace first vowel with an asterisk
```

```
s = "¥1000"
s.each_char { |x| print "#{x} " } # Prints "¥ 1 0 0 0". Ruby 1.9
0.upto(s.size-1) { |i| print "#{s[i]} " }  # Inefficient with multibyte chars
```

```
words = %w[this is a test]  # Same as ['this', 'is', 'a', 'test']
```

```
a = [1, 1, 2, 2, 3, 3, 4]
b = [5, 5, 4, 4, 3, 3, 2]
a | b  # [1, 2, 3, 4, 5]: duplicates are removed
b | a  # [5, 4, 3, 2, 1]: elements are the same, but order is different
a & b  # [2, 3, 4]
b & a  # [4, 3, 2]
```

- equality

- `equal?` method is defined by Object to test whether two values refer to exactly the same object

- `==` in Object is just alias to equal? but most class redefine it

- `!=` simple use `==` and inverts the result

- `eql?` strict version of `==`

```
1 == 1.0    # true: Fixnum and Float objects can be ==
1.eql?(1.0) # false: but they are never eql!
```

- `===` most use in case statement and:
  - Many classes, is the same as ==
  - Range to test whether a value falls within the range
  - Regexp to test whether a string matches the regular expression
  - Class to test whether an object is an instance of that class
  - In Ruby 1.9 Symbol to return true if the righthand operand is the same symbol as the left or if it is a string holding the same text

```
(1..10) === 5  # true: 5 is in the range 1..10
/\d+/ === "123" # true: the string matches the regular expression
String === "s"  # true: "s" is an instance of the class String
:s === "s"      # true in Ruby 1.9
```

- `=~` pattern match(like Regexp)

- `<=>`
  - Return -1 if its left operand is less than its right operand
  - Return 0 if the two operands are equal
  - Return 1 if the left operand is greater than the right operand
  - Return nil if can not meaningfull compared

- explicit

- to_s:
  return a human-readable representation of the object, suitable for end users
- inspect:
  is intended for debugging use, and should return a representation that is helpful to Ruby developers
- default inspect method, inherited from Object, simple calls to_s

- implicit
- invocation
- abbreviated

```
# splat (*)
x, y = 1, *[2, 3] # Same as x, y, z = 1, 2, 3
x, *y = 1, 2, 3   # x = 1; y = [2, 3]
x, *y = 1, 2      # x = 1; y = [2]
x, *y = 1         # x = 1; y = []
```

```
# Ruby 1.9 only
*x, y = 1, 2, 3   # x = [1, 2]; y = 3
*x, y = 1, 2      # x=[1]; y = 2
*x, y = 1         # x = []; y = 1

x, y *z = 1, *[2, 3, 4] # x = 1;y = 2; z=[3,4]
```

- `&&` has higher precedence that `||`
```
1 || 2 && nil  # => 1
```

- and, or and not operators are low-precedence versions of &&, ||, and !
they have lower precedence that the assignment operator

- and and or have the same precedence and not is just slightly higher
```
x || y && nil   # && is performed first   => x
x or y and nil  # evaluated left-to-right => nil
```

- `?:` operator is right-associative(same as **)
```
a ? b : c ? d : e     # This expression ..
a ? b : (c ? d : e)   # is evaluated like this..
(a ? b : c) ? d : e   # NOT like this
```

- The loop variable or variables of a for loop are not local to the loop; they remain defined even after the loop exit. Similarly, new variables defined within the body of the loop continue to exist after the loop exits.

```
squares = [1,2,3].collect {|x| x*x}   # => [1,4,9]
evens = (1..10).select {|x| x%2 == 0} # => [2,4,6,8,10]
odds = (1..10).reject {|x| x%2 == 0}  # => [1,3,5,7,9]
```

```
data = [2, 5, 3, 4]
sum = data.inject { |sum, x| sum + x }     # => 14    (1+5+3+4)
floatprod = data.inject(1.0) { |p,x| p*x } # => 120.0 (1.0*2*5*3*4)
max = data.inject { |m, x| m>x ? m : x }   # => 5
```

- implicit
- fundamental

- use return in a block will cause the method that use yield to call the block exit

- return value in block should use next

- blocks defined a new variable scope: variables created within a block exist only within that block and are undefined outside of the block

- the local variables in a method are available to any blocks within that method. so if a block assigns a value to a variable that is already defined outside of the block, this does not create a new block-local variable but instead assigns a new value to the already-existing variable.

- invocation

- an important difference between block parameters and method parameters is that block parameters are not allowed to have default values assigned as method parameters are.(seems not any more)

```
[1,2,3].each {|x,y=10| print x*y }  # SyntaxError!
```

- intuitive

- return always causes the enclosing method to return. the enclosing method, also called the lexically enclosing method, is the method that the block appears inside of when you look at the source code

- note that unlike return, break never causes the lexically enclosing method to return. break can only appear within a lexically enclosing loop or within a block. using it in any other context causes a LocalJumpError

- manipulate
- represent
