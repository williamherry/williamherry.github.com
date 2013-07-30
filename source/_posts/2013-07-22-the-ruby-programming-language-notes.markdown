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

- encapsulated

- In addition to being automatically invoked by Point.new, the initialize method is automatically made private. An object can call initialize on itself, but you cannot explicitly call initialize on p to reinitialize its state.

- coerce

```
Point::NEGATIVE_UNIT_X = Point.new(-1,0)
```

- Class Instance Variables
  - An instance variable used inside a class definition but outside an instance method definition
  - Class instance variables cannot be used from instance methods

- Define Class

```
class Point
  ...
  def Point.sum(*points)
  ...
  end
end
```

```
class Point
  ...
  def self.sum(*points)
    ...
  end
end
```

```
class << Point
  def sum(*points)
    ...
  end
end
```

```
class Point
  class << self
    ...
  end
end
```


- Method are normally public unless they are explicitly declared to be private or protected. One exception if the initialize method, which is always implicityly private.

- Another exception is any "global" method declared outside of a class definition -- those methods are defined as private instance methods of Object.

- A public method can be invoked from anywhere -- where are no restrictions on its use.

- A private method is internal to the implementation of a class, and it can only be called by other instance methods of the class (or, as we'll see later, its subclasses).

- Private methods are implicitly invoked on self, and may not be explicitly invoked on an object.

- If m is a private method, then you must invoke it in functional style as m. You cannot write o.m or even self.m

- A protected method is like a private method in that it can only be invoked from within the implementation of a class or its subclasses. It differs from a private method in that it may be explicit invocatioin on self.

- A protected method can be used, for example, to define an accessor that allows instances of a class to share internal state with each other, but does not allow users of the class to access that state.

- Protected methods are the least commonly defined and also the most difficult to understand. The rule about when a protected method can be invoked can be more formally described as follows: a protected method defined by a class C may be invoked on an object o by a method in the object p if and only if the classes of o and p are both subclasses of, or equal to, the class C.

- One of the important things to understand about object-oriented programming and subclassing is that when methods are invoked, they are looked up dynamically so that the appropriate definition or redefinition of the method is found. That is, method invocations are not bound statically at the time they are parsed, but rather, are looked up at the time they are executed.

- If you use super as a bare keyword -- with no arguments and no parentheses -- then all of the arguments that were passwd to the current method are passed to the superclass method.

- Ruby's instance variables are not inherited and have nothing to do with the inheritance mechanism. The reason that they sometimes appear to be inherited is that instance variables are created by the methods that first assign values to them, and those methods are often inherited or chainned.

```
class Point3D < Point
  def initialize(x,y,z)
    super(x,y)
    @z = z;
  end

  def to_s
    "(#@x, #@y, #@z)"
  end
end
```

- In this code, Point3D defines an initialize method that chains tothe initialize method of its superclas.. The chained method assigns values to the variables @x and @y, which makes those variables come into existence for a particular instance of Point3D

- Class variables are inherited

```
class A
  @@value = 1
  def A.vaue; @@value; end
end
print A.value
class B < A; @@value = 2; end
print A.value
class C < A; @@value = 3; end
print B.value
```

- The important difference between constants and methods is that constants are looked up in the lexical scope of the place they are used before they are looked up in the inheritance hierarchy

- Another way that new objects come into existence is as a result of the dup and clone methods. These methods allocate a new instance of the class of the object on whhich they are invoked. They then copy all the instance variables and the taintedness of the receiver object to the newly allocated object.

- clone takes this copying a step further that dup -- it also copies singleton methods of the receiver object and freezes the copy object if the original is frozen.

- A third way that objects are created is when Marshal.load is called to re-create object previously marshaled (or "serialized") with Marshal.dump.

- A singleton is a class that has only a single instance

- A singleton method is a method added to a single object rather than to a class of objects

- A module cannot be instantiated, and it cannot be subclassed

- Modules are used as namespeces and as mixins

- Just as a class object is an instance of the Class class, a module object is an instance of the Module class. Class is a subclass of Module. This means that all classes are modules, but not all modules are classes.

- Class can be used as namespaces, just as modules can. Class cannot, however, be used as mixins.

- The inclusion of a module affects the type-checking method is_a? and the switch-equality operator ===. For example, String mixes in the Comparable module and, in Ruby 1.8, also mixes in the Enumerable module:

```
"text".is_a? Comparable  # => true
Enumerable === "text"    # => true in Ruby 1.8, false in 1.9
```

- Note that instance_of? only checks the class of its receiver, not superclasses or modules, so the following is false:

```
"text".instance_of? Comparable  # => false
```

- Although every class is a module, the include method does not allow a class to be included within another class. The arguments to include must be modules declared with module, not classes

- If you want to create a module like Math or Kernel (after included, call method without module name), define you methods as instance methods of the module. Then use module_function to convert those methods to "module functions."

- In addition to loading source code, require can also load binary extensions to Ruby.

- load expects a complete filename including an extension. require is usually passed a library name, with no extension, rather than a filename. In that case, it searches for a file that has the library name as its base name and an appropriate source or native library extension. If a directory contains both an .rb source file and a binary extension file, require will load the source file instead of the binary file.

- load can load the same file multiple times. require tries to prevent multiple loads of the same file. require keeps track of the files that have been loaded by appending them to the global array $" (also known as $LOADED_FEATURES). load does not do this.

- load loads the specified file at the current $SAFE level. require loads the specified library with $SAFE set to 0, even if the code that called require has a higher value for that variable. In theory, therefore, it should be safe for require to load files with a reduced $SAFE level.

- Files loaded with load or require are executed in a new top-level scope that is different from the one in which load or require was invoked. The loaded file can see all global variables and constants that have been defined at the time it is loaded, but it does not have access to the local scope from which the load was initiated

- The local variables defined in the scope from which load or require is invoked are not visible to the loaded file.
- Any local variables created by the loaded file are discarded once the load is complete; they are never visible outside the file in which they are defined.
- At the start of the loaded file, the value of self is always the main object, just as it is when the Ruby interpreter starts running. That is, invoking load or require whthin a method invocation does not propagate the receiver object to the loaded file.
- The current module nesting is ignored within the loaded file. You cannot, fir example, open a class and load a file of method definitions. The file will be precessed in a top-level scope, not inside any class or module.

- Ruby method lookup or method name resolution ( o.m )

  - First, it checkes the eigenclass of o for singleton methods name m.
  - If no method m is found in the eigenclass, Ruby searches the class of o for an instance method named m.
  - If no method m is found in the class, Ruby searches the instance methods of any modules included by the class of o. If that class includes more that one module, then they are searched in the reverse of the order in which they were included. That is the most recently included module is searched first.
  - If no instance method m is found in the class of o or in its modules, then the search moves up the inheritance hierarchy to the superclass. Steps 2 and 3 are repeated for each class in the inheritance hierarchy until each ancestor class and its included modules have ben searched.
  - If no method named m is found after completing the search, then a method named method_missing is invoked instand. In order to find an appropriate definition of this method, the name resolution algorithm start over at step 1. The Kernel module a default implementation of method_messing, so this second pass of name resolution is guranteed to succeed.

- Class objects are special: they have superclasses.
- The eigenclasses of class objects are also special: they have superclasses, too.

- refection
- introspection
- examine
- malicious

- Note that eval evaluates its code in a temporary scope. eval can alter the value of instance variables that already exist. But any new instance variables it defines are local to the invocation of eval and cease to exist when it returns. (It is as if the evaluated code is run in the body of a block -- variables local to a block do not exist outside the block)

- synonym

- It is important to understand that define_method is private. You must be inside the class or module you want to use it on in order to call it:

```
# Add an instance method named m to class with body b
def add_method(c, m, &b)
  c.class_eval {
    define_method(m, &b)
  }
end

add_method(String, :greet) { "Hello, " + self }

"world".greet # => "Hello, world"
```


