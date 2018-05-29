---
title: "Introduction to TensorFlow Variables"
description: "A gentle introduction to the basics of TensorFlow variables."
tags: [ "Python", "TensorFlow", "Machine-Learning" ]
lastmod: 2018-05-29
date: "2018-05-29T03:30:18"
categories:
  - "Python"
  - "TensorFlow"
disable_comments: true
---

Hello there, today we are going to learn about TensorFlow variables.

### What are Tensorflow Variables?
TensorFlow variables, just like Python variables, are used to store, reference and manipulate data. Although functionally, they are very different. This article will try to help you understand what a TensorFlow variable is and the things that we need to know before these variables can process information.


### Creating Tensorflow Variables
Creating and assigning variables in Python is really easy:

```py
In [1]: x = 5
In [2]: y = x * 3
In [3]: print(y)
15
```

Easy, right? Here's a catch though -- the Tensorflow variables initialization syntax is a _little_ different than this. In TensorFlow, you need to create a variable by using the [tf.Variable()][1] constructor. It takes the variable initialization value as a parameter and has an optional keyword parameter to assign a internal name to the variable, different than the actual variable name that you define. This name comes in handy when you need to save and restore your variable instances while using TensorFlow.

We’ll talk more about variable naming in a different post, maybe. Meanwhile, here’s what the variable constructor looks like:

```py
In [4]: import tensorflow as tf
In [5]: w = tf.Variable(<initial-value>, name=<optional-name>)
```

Alright, let’s try creating a Tensorflow variable:

```py
In [6]: x = tf.constant(5, name='x')
In [7]: y = tf.Variable(x * 3, name='y')
```

Here, we’ve created a [tf.constant][2] with a value ‘5’. We take that constant and use it to compute a variable y. Although quite the same thing, this doesn't look similar to the variables that we created before.

What is the difference though? Let’s try printing the value of the computed variable:

```python
In [8]: print(y)
Tensor("y/read:0", shape=(), dtype=int32)
```

As you can see, a TensorFlow variable does not compute any values when initialized. This is because TensorFlow handles variables differently.

###### In what ways is a Tensorflow variable different?

Unlike Python variables, there is no computation taking place at runtime. Instead, TensorFlow takes as input all the variables and constants as functions, and stores them as tensors. Using these tensors, Tensorflow creates a graph which is then used to compute the output at the time of session execution.

###### What is a computational graph?
A computational graph is a group or series of related TensorFlow operations. Consider it as a data structure graph with nodes connected to each other through links, where each node may take zero or more tensors as input. Each connected node produces a tensor as an output and passes it up ahead to the next connected node as an input. This continues until it reaches the root node, or till the end of the computational graph.

###### What are tensors?
Tensors are nothing but objects that describe relations between vectors and other Tensors. Tensors represent a computational output of any operation. From the documentation:

>"A Tensor is a symbolic handle to one of the outputs of an Operation. It does not hold the values of that operation's output, but instead provides a means of computing those values in a TensorFlow [tf.Session()][3]"

Simply put, tensors are a structure which do not hold any computational value. Instead, they contain information on how to compute the given values, or basically states the flow of tensors (_\*slow claps\*_). To actually transform these tensors into a graph (or graphs), we need to initialize the variables, which creates a tensor object of the graph. A tensor object is nothing but a set of operations which may have tensors at input and output.

Multiple such related tensor objects make up a computational graph. We then execute this graph in a session to calculate the value for the initialized variables, and this session needs to be executed each time the value of the variables is to be updated.

### Generating a Tensorflow computational graph

We want to generate a graph for all the variables in the code. To do that, we use [tf.global_variables_initializer()][4]. This is a TensorFlow helper function which initializes all the variables by creating a graph of dependencies and relationships between the variables when executed.<sup>[1](#references)</sup> There also exists an alternative if you want to initialize only a select bunch of variables.<sup>[2](#references)</sup>

Okay, now on with initializing the variables:

```py
In  [9]: import tensorflow as tf
In [10]: x = tf.constant(5, name='x')
In [11]: y = tf.Variable(x * 3, name='y')
In [12]: compute = tf.global_variables_initializer()
```

The code doesn’t actually compute anything yet. This is because we have only initialized the graph so far.

Let us check what compute contains:

```py
In [13]: print(compute)
name: "init"
op: "NoOp"
input: "^Variable/Assign"
input: "^y/Assign"
```

Although the output doesn't really make much sense, we have generated the computational graph by initializing the variables. Now to make TensorFlow compute the value for our operation, we need to execute the computational graph and the variable y.

### Executing the TensorFlow computational graph

To execute a graph, we first need to create a TensorFlow session wherein the output for the initialized variables can be computed. A session in TensorFlow is an instance for executing initialized variables. From the documentation:

>"A Session instance encapsulates the environment in which Operations in a Graph are executed to compute Tensors."

We can create a session with [tf.Session()][3] and use it to execute the graph. We also need to explicitly close the session after we’re done using it. A workaround to closing the session is to create a session using the [with](https://docs.python.org/3/reference/compound_stmts.html#with) statement in Python. Then, we can compute the graph and execute the variable y. 


```py
In[14]: with tf.Session() as sess:
            sess.run(compute)
            print(sess.run(y))
15
```

_**Do we need to do this every time we create a session?** Yes, and no._

We need to execute the variable every time it is updated, but we initialize the variables just once. It is not recommended to reinitialize variables because doing so generates duplicate operations.<sup>[3](#references)</sup>

**_Can we update the variables in the session?_** _Yes, that can be done._

Let’s try it out for the sake of completeness:

```py
In [15]: import tensorflow as tf
In [16]: a = tf.Variable(0, name=‘a’)
In [17]: init = tf.global_variables_initializer()
In [18]: with tf.Session() as sess:
            sess.run(init)
            for i in range(5):
                a = a + i
                print(sess.run(a))
0
1
3
6
10
```

The variables can be updated during the session, and each time the variable is updated, we need to run it to compute the updated value. Once again, we do not need to initialize the variables every time we run the operations. This is because every time we run tf.global_variable_initializer() a duplicate graph for the same operation is initialized.


So that's it for this article, I hope this brief introduction to TensorFlow variables helped you understand the basic functionality of a TensorFlow variable.

##### References

###### Citations:

[[1]] : [TensorFlow Helper Functions.](https://www.tensorflow.org/versions/r0.12/api_docs/python/state_ops/variable_helper_functions)

[[2]] : [TensorFlow Variables initializer: tf.variables_initializer.](https://www.tensorflow.org/versions/master/api_docs/python/tf/variables_initializer)

[[3]] : [KDNuggets: How not to program TensorFlow Graphs.](http://www.kdnuggets.com/2017/05/how-not-program-tensorflow-graph.html)


###### Formal References:

TensorFlow documentation : [Variables: Creation, Initialization, Saving, and Loading](https://www.tensorflow.org/programmers_guide/variables)

[//]: # (URL's referenced in the page)
[1]: https://www.tensorflow.org/api_docs/python/tf/Variable "TensorFlow variable"
[2]: https://www.tensorflow.org/api_docs/python/tf/constant "TensorFlow constant"
[3]: https://www.tensorflow.org/api_docs/python/tf/Session "Tensorflow Session"
[4]: https://www.tensorflow.org/versions/master/api_docs/python/tf/global_variables_initializer "Tensorflow Global Variables Initializer"


