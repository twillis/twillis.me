Title: Achievement Unlocked
Author: Tom Willis
published_date: 2013-06-01
teaser: 
    <p>How I overcome analysis paralysis.</p>



Test driven development or
[TDD](http://en.wikipedia.org/wiki/Test-driven_development) is one of
those things that I'm constantly figuring out where it's valuable and
where it's not. I often find it more valuable at the early stages of a
project or feature. If nothing else, it makes developing the project
or feature a game which in itself helps to me to keep going.

## Level 1

Once I have made the decision to develop a project or feature, writing
a failing test and seeing it fail is what I try to do as quickly as I
can. To me, it represents my starting of the doing and in some sense
the end of my planning. The failing test can be as simple as ...


    :::python

    import unittest
    class TestNewFeature(unittest.TestCase):
        def testFail(self):
            self.fail("YAY")

The act of getting this to run and fail represents so many things. For
example, it proves I can run python properly. It also means I have
chosen a place for the file to live. It represents my decision on how
I track my progress in developing this new feature. In other words. I
now know how to start. All these little micro decisions might take
place in a matter of seconds in an existing project, but for a new
project, the number of decisions made that are represented are even
greater. 

For a new project it represents that I have at least initially chosen
which version of the language I am going to support, after all, you
have to support at least one. It likely also represents that I have
made the decision to either use virtualenv or buildout depending on
the anticipated complexity of the project, and that I have it setup
properly, and my editor is working with it correctly. At this point I
probably will also create the Git repository that the project will
live in, configure the .gitignore perhaps even make that initial
commit. 

All of this might take more than a minute or two but it's work that
has to be done that will help me later on. 

So what's next?

## Level 2

Often times I have a rough idea of the overall features, but unit
tests are supposed to test one and only one thing. A feature of any
moderate complexity has to be broken down into smaller pieces. The
next step for me will likely be writing failing tests for all those
things that I am anticipating will make the feature. This can be a
daunting task in itself, but I think it helps to feel comfortable
writing code, running that code and perhaps even deleting that code.  

### Can I?

Lately I've been writing the initial tests in the form of
testCanI. Recently I started writing a static site generator. My
thought was I could somehow coax pyramid into rendering templates with
some standard context. The templates would have to be located
according to some convention, but most of all generating the site
would first require a listing of all the template files. So, my first
test was "Can I get a list of pages?" or ...


    :::python
    import unittest

    class TestPages(unittest.TestCase):
          def testCanIGetPageList(self):
	      # http get /list
              self.fail("no, you have to implement this....")


The very act of writing this test represents a decision on what to do
next. Getting it to pass requires a whole other series of decisions
that have to be made and verified. For one, I have to figure out how
to pass an HTTP request to the application and verify the result. one
of the asserts will surely be that I get a valid response. I decide a
valid response is that the status code of the response is 200. But
wait, the application doesn't even exist. What should it be named?
What framework do I use? Well first things first, Can I get a valid
response?


    :::python
    import unittest

    class TestPages(unittest.TestCase):
          def testCanIGetValidResponse(self):
              self.assertEqual(response.status, 200)

          def testCanIGetPageList(self):
	      # http get /list
              self.fail("no, you have to implement this....")


One should note that it is a pretty good idea to really know your test
runner and you should feel comfortable doing things like switching
from running individual tests or a subset of tests, to running the
whole test suite. It helps me stay focused on what I'm trying to
accomplish right now. Ideally, the time spent between implementing the
code you are testing and the actual testing of that code should be as
quick as is reasonably possible.


## What's it all mean?

Hopefully I've demonstrated how I get started on development work
through the above examples. In a sense what I'm doing is making an
executable to do list. That is a planning exercise, your test
runner will automatically tell you how far along you are and when you
are done. A software developer after all, should be more than
qualified to make their computer help them make their work easier. And
we as software developers should hopefully be striving for new and
better ways to make our jobs of developing easier. TDD is a great way
to do that.

It's probably a good idea to mention that even though I find this
valuable and I would guess any developer would find it valuable once
they are used to it, the act of doing it this way is not going to
directly add any perceived business value to the stakeholders. I
believe maintaining a unit test suite is more an advantage to the
individual that uses it to yield useful code, the stakeholders on the
other hand will only value the useful code, not the means of getting
it. As a result, I tend to not buy into the religion of TDD that
insists on an ideal of 100% code coverage and testing being part of
the development process.

So do TDD for you, yield working code for your stakeholders using TDD
as a means to an end.