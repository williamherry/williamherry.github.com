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
not need any more

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

- An important feature of procs and lambdas is that they are closures: they retain access to the local variables that were in scope when they were defined, even then the proc or lambda is invoked from a different scope

- subtle
- invocation
- respectively

- Ruby implementations typically treat Fixnum and Symbol values as immediate values rather than as true object references. For this reason, singleton methods may not be defined on Fixnum and Symbol objects. For consistency, singletons are also prohibited on other Numberic objects.

- Interestingly, undef can be used to undefine inherited method, without affecting the definition of the method in the class from which it is inherited.

- It cannot be used to undefine a singleton method in the way that def can be used to define such a method.

- punctuation
- unary

```
alias aka also_known_as   # alias new_name existing_name
```

```
def hello
  puts 'Hello World'
end

alias original_hello hello

def hello
  puts 'Your attention please'
  original_hello
  puts 'This has been a test'
end
```

- overload
- ambiguous
- wrinkle

```
def suffix(s, index=s.size-1)
  s[index, s.size-index]
end
```

- In Ruby 1.8, method parameters with default values must appear after all ordinary parameters in the parameter list. Ruby 1.9 relaxes this restriction and allows ordinary parameters to appear after parameters with defaults. It still requires all parameters with defaults to be adjacent in the parameter list - you can't declare two parameters with default values with an ordinary parameter between them, for example.

- No more than one parameter may be prefixed with an *. In Ruby 1.8, this parameter must appear after all ordinary parameters and after all parameters with defaults specified. It should be the last parameter of the method, unless the method also has a parameter with an & prefix. In Ruby 1.9, a parameter with an * prefix must still appear after any parameters with defaults specified, but it may be followed by additional ordinary parameters. It must also still appear before any &-prefixed parameter.

- anonymity

- Blocks are syntactic structures in Ruby; they are not objects, and cannot be manipulated as objects.
- It is possible, however, to create an object that represents a block. Depending on how the object is created, it is called a proc or a lambda
- Procs have block-like behavior and lambdas have method-like behavior. Both, however, are instances of class Proc.

- Proc.new => procs
- Kernel.lambda => lambda
- Kernel.proc => alias of lambda in Ruby 1.8
- Kernel.proc => alias of Proc.new in Ruby 1.9

```
succ = lambda {|x| x+1}  # Ruby 1.8 lambda
succ = ->(x){ x+1 }      # Ruby 1.9 lambda
```

```
# This lambda takes 2 args and declares 3 local vars
f = ->(x,y; i,j,k) { ... }
```

```
zoom = ->(x,y,factor=2) { [x*factor, y*factor] }   # only on Ruby 1.9
```

```
succ = ->x { x+1 }
f = -> x,y; i,j,k { ... }
zoom = ->x,y,factor=2 { [x*factor, y*factor] }
```

- Proc Equality - The `==` method only returns true if on Proc is a clone or duplicate of the other;

```
p = lambda {|x| x*x }
q = p.dup
q == p                      # => true: the two procs are equal
p.object_id == q.object_id  # => false: they are not the same object

- A proc is the object form of a block, and it behaves like a block.
- A lambda has slightly modified behavior and behaves more like a mthod that a block.

- The return statement in a block does not just return from the block to the invoking iterator, it returns from the method that invoked the interator.

```
def test
  puts "entering method"
  1.times { puts "entering block"; return }  # Makes test method return
  puts "exiting method"   # This line is never executed
end
test
```

- A proc is like a block, so if you call a proc that executes a return statement, it attempts to return from the method that encloses the block that was converted to the proc.

```
def test
  puts "entering method"
  p = Proc.new { puts "entering proc"; return }
  p.call                # Invoking the proc makes method return
  puts "exiting method" # This line is never executed
end
test
```

- Using a return statement in a proc is tricky, however, because procs are ofter passed around between methods. By the time a proc is invoked, the lexically enclosing method may already have returned:

```
def procBuilder(message)            # Create and return a proc
  Proc.new { puts message; return } # return returns from procBuilder
  # but procBuilder has already returned here!
end

def test
  puts "entering method"
  p = procBuilder("entering proc")
  p.call                # Prints "entering proc" and raises LocalJumpError!
  puts "exiting method" # This line is never executed
end
test
```

- A return statement in a lambda, therefore, returns from the lambda itself, not from the method that surrounds the creation site of the lambda:

```
def test
  puts "entering method"
  p = lambda { puts "entering lambda"; return }
  p.call                  # Invoking the lambda does not make the method return
  puts "exiting method"   # This line *is* executed now
end
test
```

- The fact that return in lambda only returns from the lambda itself means that we never have to worry about LocalJumpError

```
def lambdaBuilder(message)            # Create and return a proc
  lambda { puts message; return }     # return returns from lambda
  # but procBuilder has already returned here!
end

def test
  puts "entering method"
  p = lambdaBuilder("entering lambda")
  p.call                # Prints "entering lambda"
  puts "exiting method" # This line is executed
end
test
```

- A top-level next statement works the same in a block, proc, or lambda: ti causes the yield statement or call method that invoked the block, proc, or lambda to return. If next is followed by an expression, then the value of that expression becomes the rturn value of the block, proc, or lambda.

- redo also works the same in procs and lambdas: it transfers control back to the beginning of the proc or lambda.

- retry is never allowed in procs or lambdas: using it always results in a LocalJumpError.

- raise behaves the same in blocks, procs, and lambdas. Exceptions always propagate up the call stack. If a block, proc, or lambda raises an exception and there is no local rescue clause, the exception first propagates to the method that invoked the block with yield or that invoked the proc or lambda with call.

- Argument passing to procs and lambdas
  - The yield statement uses yield semantics
  - method invocation uses invocation semantics
  - Yield semantics are similar to parallel assignment
  - invoking a proc uses yield semantics
  - invoking a lambda uses invocation semantics

```
p = Proc.new {|x,y| print x,y }
p.call(1)       # x,y=1:     nil used for missing rvalue:  Prints 1nil
p.call(1,2)     # x,y=1,2:   2 lvalues, 2rvalues:          Print 12
p.call(1,2,3)   # x,y=1,2,3: extra rvalues discarded:      Print 12
p.call([1,2])   # x,y=[1,2]: array automatically unpacked: Print 12
```

```
l = lambda {|x,y| print x,y }
l.call(1,2)     # This works
l.call(1)       # Wrong number of arguments
l.call(1,2,3)   # Wrong number of arguments
l.call([1,2])   # Wrong number of arguments
l.call(*[1,2])  # Works: explicit splat to unpack the array
```

- In Ruby, procs and lambdas are closures.
- An object that is both an invocable function and a variable binding for that function.
- When you create a proc or a lambda, the resulting Proc object holds not just the executable block but also bindings for all the variables used by the block.

- One important difference between Method objects and Proc objects is that Method objects are not closures.
