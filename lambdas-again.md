---
title: "Lambdas again"
tags: elm syntax 
date: 2021-01-30
---

Just after I [wrote about lambdas in Elm](/posts/2021-01-24-lambdas) suggesting that they were not particularly useful and if we had a mind to we could remove them from the Elm language (very unlikely to happen since I think it would cause too much anger within the Elm community), I read on the [awesome Python newsletter](https://Python.libhunt.com/newsletter) a post regarding lambdas in Python, specifically [5 uses of lambda expressions](https://medium.com/techtofreedom/5-uses-of-lambda-functions-in-Python-97c7c1a87244). The author doesn't quite have a main point, but it seems to be that lambda expressions are useful and you should use them in your Python code.

To be honest, I didn't find their use-cases terribly convincing. I'll go through them briefly here and explain why I'm a little sceptical, but I'll finish with a major difference between lambdas in Python and lambdas in Elm (or any other functional language). I'll state right now, that I'm not sure whether this distinguishing point is a point in favour of lambdas in Python or not.

## 5 uses of lambda expressions in Python

So I'm going to go through the use-cases as stated in the [linked article](https://medium.com/techtofreedom/5-uses-of-lambda-functions-in-Python-97c7c1a87244).

### Use 1: Give It a Name and Use It As Normal

> "If we just need a simple function, lambda is a good choice, since it can be treated as a simpler way to define a function."

```Python
lambda_add_ten = lambda x: x + 10
```

I'm not sure why the author considers this simpler than:


```Python
def lambda_add_ten (x):
    return x + 10
```

I guess you can do it on a single line, and I guess it uses one fewer key words. But personally you're just making up a second way to achieve the same means which nearly always *inhibits* understanding rather than *enhances* it. 

### Use 2: Cooperate With Higher Order Functions

This is perhaps the most common use case in functional programming as well. For the same reasons I'm a little sceptical that this is useful in Elm, I'm perhaps even more sceptical it's useful in Python. In Python you're *always* able to just add a definition above the use case, you do not need to create a *let/in* block.

The precise example the author gives is:

```Python
numbers = [1, 12, 37, 43, 51, 62, 83, 43, 90, 2020]

print(list(filter(lambda x: x % 2 == 1, numbers)))
```

Which he compares favourably with a version that doesn't use higher-order functions. But that is not the relevant comparison to make. The relevant comparison is one that doesn't use a lambda. Before you read my version, can you quickly discern what the above fragment is doing? Now try my fragment:


```Python
numbers = [1, 12, 37, 43, 51, 62, 83, 43, 90, 2020]

def is_odd (x) :
    return x % 2 == 1
print(list(filter(is_odd, numbers)))
```

I think my version is better. The author even admits that in Python a better way to do this is with list-comprehensions. That's a discussion for another day, but still it's not helpful to the author's main point.

### Use 3: Assign to the “Key” Argument

This is sort of just another case of the above, the author's example is:

```Python
leaders = ["Warren Buffett", "Yang Zhou", "Tim Cook", "Elon Musk"]
leaders.sort(key=lambda x: len(x))
```

In this case the `sort` call can simply be written as `leaders.sort(key=len)` as the `len` variable is **already** a named function. If the function was not already named, I would give it a name just as in the *Use 2* case above, and prefer it for the same reasons.


### Use 4: Immediately Invoke It

From the article:

> "An immediately invoked function expression (IIFE) is an idiom from JavaScript. The lambda functions in Python support this trick as well. We can immediately run a lambda function as the following:"

```elm
>>> (lambda x,y:x*y)(2,3)
6
```

Why would anyone ever wish to do this? I'm at a loss.


### Use 5: Apply It in Closures

This case is talking about functions which return a function. Here is the non-lambda version:

```Python
def outer_func():
    leader = "Yang Zhou"
    def print_leader(location=""):
        return leader + " in the " + location
    return print_leader
```

and here is the version using a lambda: 

```Python
def outer_func():
    leader = "Yang Zhou"
    return lambda location="": leader + " in the " + location
```

In this case I could just about see someone preferring the latter version. I think they are much of a muchness personally. In addition, such functions are generally a bit more complicated than this. I think any gain in the fact that the lambda version is 'shorter' is lost by the fact that it's just another way to define a function. But of the five use cases this is certainly the most convincing. For me, it's still not worth it, but you can make up your own mind.

Also I would say the conclusion here is that Python could have a nicer syntax for definining functions that return other functions. Something like:


```Python
def outer_func()(location:""):
    leader = "Yang Zhou"
    return leader + " in the " + location
```

A final small point, I'm not sure why `location` is defaulted to `""` since that makes the sentence returned non-grammatical, not to mention that because the original *'function'* had no arguments the whole 'closure' was not necssary, but I guess this is just an example.

## How are lambdas different in Python and Elm

Okay so at least according to the five use-cases identified in the original post, I'm still not seeing a great need for lambdas in Python, just as I don't see them as particularly necessary in Elm. But they are a bit different. The distinguishing feature is that in Elm a lambda expression really is just a short hand for a `let` defined function with a name. In Python however, defined functions must use the `return` keyword to actually return something, whereas using a lambda the result of evaluating the expression is automatically the returned result of the function.

I'm not sure whether this suggests they are more or less of an enhancement in Python in than in functional languages. On the one hand they are clearly introducing something new. On the other hand, it introduces a different behaviour that I can see could being a source of bugs, particularly in a dynamically typed language. Consider the first example in this article, the use of lambda expressions to define a named function:


```Python
lambda_add_ten = lambda x: x + 10
```

I suggested that I prefer the following:


```Python
def lambda_add_ten (x):
    return x + 10
```

Even if you do not prefer that, I think it's hard to argue that the first is **much** better than the second and hence really justifies complicating the language for. Anyway, it would be easy to make the following mistake when using the second syntax, here I've just mistakenly missed off the `return` keyword:


```Python
def lambda_add_ten (x):
    x + 10
```

A decent linter probably picks this up, but it is valid Python, and if the returned expression was some side-effecting function call then the linter wouldn't even necessarily pick it up. Anyway, I think this sort of mistake is easier to make if you've just been using lambda expressions elsewhere. Therefore I think that the inclusion of lambdas in the language make this mistake easier and more common.

To sum up, I'm still yet to be convinced that either Elm or Python really needs to include lambda expressions, I just don't see much benefit. But this is not a hill I'm going to die on, for the most part I can simply ignore them, and they certainly don't make code **much** harder to comprehend if someone else uses them. So I don't think they add much benefit, but neither are they too determintal, and for that reason I would strongly expect them to remain parts of both languages and newer languages as well.
