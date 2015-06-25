---
layout: post
title:  "Python Study Notes"
date:   2015-06-24
categories: Study-Notes
excerpt: 
comments: true
---

* content
{:toc}

## Python discussion

[Link](https://www.udemy.com/blog/python-vs-java/)

1. Dynamic vs Static Typing

One of the biggest differences between Python and Java is the way that each language handles variables. Java forces you to define the type of a variable when you first declare it and will not allow you to change the type later in the program. This is known as static typing. In contrast, Python uses dynamic typing, which allows you to change the type of a variable, by replacing an integer with a string, for example.
Dynamic typing is easier for the novice programmer to get to grips with, because it means you can just use your variables as you want to without worrying too much about their types. However, many developers would argue that static typing reduces the risk of undetected errors plaguing your program. When variables do not need to be explicitly declared before you use them, it is easy to misspell a variable name and accidentally create a whole new variable.

2. Braces vs Indentation

Python is unusual among programming languages in that it uses indentation to separate code into blocks. Java, like most other languages, uses curly braces to define the beginning and end of each function and class definition. The advantage of using indentation is that it forces you to set your program out in a way that is easy to read, and there is no chance of errors resulting from a missing brace.
You can learn more about the unique features of Python in The Ultimate Python Programming Tutorial. This course will teach you to create clear, efficient code, as well as how to debug your applications after writing them.

3. Speed vs Portability

The great advantage of Java is that it can be used to create platform-independent applications. Any computer or mobile device that is able to run the Java virtual machine can run a Java application, whereas to run Python programs you need a compiler that can turn Python code into code that your particular operating system can understand. Thanks to the popularity of Java for web applications and simple desktop programs, most devices already have the Java virtual machine installed, so a Java programmer can be confident that their application will be usable by almost all users. The disadvantage of running inside a virtual machine is that Java programs run more slowly than Python programs.

## DJango

### Template

When you subclass a class-based view, you can override attributes (such as the template_name) or methods (such asget_context_data) in your subclass to provide new values or methods. Consider, for example, a view that just displays one template, about.html. Django has a generic view to do this - TemplateView - so we can just subclass it, and override the template name:

~~~ python
# some_app/views.py
from django.views.generic import TemplateView

class AboutView(TemplateView):
    template_name = "about.html"
~~~

Then, we just need to add this new view into our URLconf. As the class-based views themselves are classes, we point the URL to the as_view class method instead, which is the entry point for class-based views:

~~~ python
# urls.py
from django.conf.urls import patterns, url, include
from some_app.views import AboutView

urlpatterns = patterns('',
    (r'^about/', AboutView.as_view()),
)
~~~

### reverse

reverse()

If you need to use something similar to the url template tag in your code, Django provides the following function:
reverse(viewname[, urlconf=None, args=None, kwargs=None, current_app=None])

viewname can be a string containing the Python path to the view object, a URL pattern name, or the callable view object. For example, given the following url:

~~~ python
url(r'^archive/$', 'news.views.archive', name='news_archive')
~~~

you can use any of the following to reverse the URL:

~~~ python
# using the Python path
reverse('news.views.archive')

# using the named URL
reverse('news_archive')

# passing a callable object
from news import views
reverse(views.archive)
~~~
