---
draft: false
title: If A Task Can Be Automated, Automate It
description: >-
  Why not automate a task you can clearly automate and save yourself some time?
date: "2023-04-27T05:36:31+06:00"
tags: [ "automation", "cli" ]
categories: [ "story" ]
---

## TLDR

I automated a task that I could clearly automate. I could've easily done it
manually but why not save myself some time and automate it?

## Introduction

So here I was, going on about my day as usual, downloading wallpapers from the
internet. It was then when I realized that all the downloaded files were the
same name. I had to rename them manually to add them to my existing collection.
Here's where the idea to write a script for it comes into mind.

I have my wallpaper names formatted in a specific way. All the files are named
as random integers between a certain range and the appropriate file extension.

## Steps To Take

Here's what my script needed to accomplish.

- List all the files in the directory with all the images.
- Generate a random integer within a specific range.
- Add the original file extension to the generated integer.
- Rename the file with the generated name.

Now that I had my goals set, it was time to get to coding. I once again reached
out to Python for the job as it seems to be the most convenient tool for the
job.

```python
def main():
    pass

if __name__ == "__main__":
    main()
```

## Get The Files To Rename

With that base template in place, let's code the main function. The first thing
it needs to do is list all the files in the directory of the images. We can take
the name of the directory as input from the user.

Python has an `os` module that we can use to list files in a directory. To use
that, we first have to import the module and call the `listdir` function with
the directory name as the first argument.

```python
# import os

directory = input("directory:: ")
files = os.listdir(directory)
```

If we were to print the contents of `files` we'd see something like this.

```none
['1.jpg', '2.jpg', '3.jpg', '4.jpg', '5.jpg']
```

## Generate A Random Integer

We now have the original file names. Let's now take an array to generate a
random integer corresponding to each file name. For that, we'll use the `random`
module and its `randint` function which takes two arguments, the lower and upper
bounds of the range.

We used an array to keep track of the numbers generated and made sure that those
numbers were unique. At the end of the loop, we're sure that we have a list of
unique random numbers that we can use to generate our file names. We're now done
with the second step of the process.

```python
# import random


random_names = []

for file in files:
    random_name = random.randint(100_000, 999_999)
    while random_name in random_names:
        random_name = random.randint(100_000, 999_999)
    random_names.append(random_name)
```

## Add The File Extension

We extracted the file extension from the original file name and appended it to
the generated random number after converting it into a string. With this, we've
completed the third step of the process.


```python
extension = file[file.rindex("."):]
new_name = str(random_name) + extension
```

## Rename The Files

And, finally, we're ready to rename the files. For that, we'll use the `rename`
function from the `os` module. It takes two arguments, the original file name
and the new file name. Since our original files were inside a directory, we have
to join the directory name before the file name with a slash.

```python
# import os

os.rename(directory + "/" + file, directory + "/" + new_name)
```

## Results

```python
import os
import random


def main():
    directory = input("directory:: ")
    files = os.listdir(directory)
    random_names = []

    for file in files:
        random_name = random.randint(100_000, 999_999)
        while random_name in random_names:
            random_name = random.randint(100_000, 999_999)
        random_names.append(random_name)

        extension = file[file.rindex("."):]
        new_name = str(random_name) + extension

        os.rename(directory + "/" + file, directory + "/" + new_name)


if __name__ == "__main__":
    main()
```

There you have it, all the necessary steps completed. We can now run the script
and if we provide the directory name as input, it'll rename all the files in
that directory with a random integer between 100,000 and 999,999.

```none
directory:: /home/username/wallpapers
```

```none
/home/username/wallpapers/1.jpg -> /home/username/wallpapers/123456.jpg
/home/username/wallpapers/2.jpg -> /home/username/wallpapers/234567.jpg
/home/username/wallpapers/3.jpg -> /home/username/wallpapers/345678.jpg
/home/username/wallpapers/4.jpg -> /home/username/wallpapers/456789.jpg
/home/username/wallpapers/5.jpg -> /home/username/wallpapers/567890.jpg
```

## Conclusion

And that's how I automated a task that I could clearly automate. I could've
easily done it manually but why not save myself some time and automate it? I
hope this post was helpful to you. If you have any questions or suggestions,
feel free to reach out to me on [Twitter](https://twitter.com/jNaimXIII).
I'll see you in the next one.
