---
title: Algorithms In Our Lives
description: "Algorithms are everywhere, you just don't notice them."
date: "2023-04-07T21:14:53+06:00"
tags:
  - algorithms
categories:
  - story
---

Our lives are all but actions performed towards achieving a particular goal. We
do many things in life; some repetitive, some unique. In trying to perform
these actions we find patterns, make connections to previous knowledge, and
utilize what we know to perform them better, faster, and more efficiently.

Algorithms, by the book, mean a set of well-defined instructions or steps that
are followed to solve a particular problem or accomplish a specific task. What
does it mean to us? How do they apply in our daily lives?

Assume you're a teacher. You're given a list of all 2,000 students in your
school. You're now trying to find a particular student from that list but
checking them one by one would take an incredibly long time. If that student
were to be found at the end of the list, you'd have to look through the
previous 1,999 students before you finally find the one you're looking for.

What you've done here is a linear search to find a particular target of yours.
Now, think about this, is there any way you can make this process a bit faster?
If this is a list of students, it must have a few things we can use to our
advantage. These lists are often ordered in an ascending order, generally
regarding the student ID number.

Think about how you'd search for your student if you knew his student ID. Would
you still follow that same approach as before? You would most definitely find
your target using the old method and ignoring all the context but, why do that
when you could find your target in a matter of seconds rather than minutes?

How might we reduce our searching time by such magnitudes you ask? We search
differently than before. As we know the ID of the student, we can first go to
the middle of the list and check if we've found them. If we have, then great,
we don't have to search anymore. The story goes differently when we don't find
them. We can now compare the ID of the student we landed on to the student
we're searching for.

If our target student has an ID less than the one we're currently looking at,
it means we've already gone past him in the list. The reason behind this is,
our list is sorted in an ascending order. If the ID is already greater than the
target ID, there's no possibility that we'll find our target later down the
list. Thus we can entirely skip the second half of the list and search the
first half for our target.

Do you see what we've essentially done here? We've cut the list in half, down
to only 1,000 students. Now, the real efficiency gains kick in. You repeat the
same process as before. Go to the middle of the newly halved list and check if
it's the student you're looking for. This time, it appears that you've found an
ID that is smaller than the target student's ID. What does that imply? We can
now be sure that our student can't be on the top half of our already halved
list of students for the same reason we were able to discard the later half of
our list in the first step of searching for the student. As our target ID is
greater than the ID we're looking at, our target student has no possibility of
being in the upper half of the list.

Thus, we can discard the top half of our previously halved list. This brings us
down to only 500 students. That's just a quarter of our original student count
when we started. The fascinating thing is that we've only looked at 2 students
so far. If you'd used the previous linear searching method, you would've only
searched 0.001% of your entire list whereas with what we've done here, we
already discarded 75% of our list. And, the remaining 25% we can reduce further
by repeating this same method.

What this ultimately means is that you've been halving your list every time and
eventually you'll find your target student. If the student exists in this list,
you'll always find it within a definite number of tries. That number of tries
is the natural log of the number of students in your list.

So, in the scenarios above, you've found your target student but you've
searched for them in vastly different manners. These manners, methods, ways, or
formulas you use to solve a specific problem, in this case finding a student,
is what's known as an algorithm.

In mathematical terms, both the above methods have a name and characteristics
that define how efficient they are at solving a specific problem. The two
methods discussed in this article were Linear Search and Binary Search
respectively. With linear search, the time required to find a specific target
grows linearly as the input size grows whereas, with binary search, you'll
require less and less time to find the target as the input size grows.

Although both are valid methods for searching for students in a list, one is
vastly superior to the other as the situation provides ways to utilize such
methods.

Algorithms can be used in any form, in any place, and at any time. It is up to
us to decide what works best for our particular use case and do the things we
do in an efficient and cost-saving manner.

---

This article was submitted for my school magazine and as I had to target a
general audience, this does not contain any technical specifications. I've
tried to explain in simple terms and a story driven manner what an algorithm is
and how we apply it in our lives.
