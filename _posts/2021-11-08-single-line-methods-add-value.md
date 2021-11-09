---
title: Single line methods add value
author: Juan Antonio Romero Molero
date: 2021-11-08 19:00:00 +0200
categories: [Backend, Frontend]
tags: [clean code]
description: "Single-line methods add value to your code, hiding more concrete implementation details and making it easier for other programmers to read."
---

## Introduction

Sometimes people may complain about how unnecessary some methods are. They say there is no need to create a new method if it is made up of a single line of code, that leaving the code in a higher-order place is enough because it is easy to understand. But I can't agree with that.

## A good enough code

Let's see an example. Imagine that we are creating a service to export a file with users and we have something like this:

```ruby
class UsersExporter

  # more code

  def export
    users = find_users_to_export

    file = File.new "files/users_#{Time.new.to_i}.csv", 'w'

    write_users(file, users)
  end

  # more code

end
```

Think about the line where we are creating the destination file. It's easy to read and any developer can read and understand that sentence in the blink of an eye. We might think that our code is good enough.

## Hiding details

But let's put this example aside for a moment. Some years ago, Uncle Bob wrote one of the most acclaimed books on software development, "Clean Code: A Handbook of Agile Software Craftsmanship".

One of the quotes that he left us in this book was the following:

> So if you want to go fast, if you want to get done quickly, if you want your code to be easy to write, make it easy to read.

As he explains, one thing we need to do is hide the implementation details in our code. We want readers to be able to know what our code does without having to stop to read it and understand everything.

Following *The Stepdown Rule* that he talks about, we want to keep our main methods as abstract as possible. It's not about really hiding the implementation details so that no one can figure out how our program works, but only needing to understand the low-level details when necessary.

## Improving the code

So, go back to our example and look carefully at the suspicious line:

```ruby
file = File.new "files/users_#{Time.new.to_i}.csv", 'w'
```

How many things can we know when we read this line?

- Use the `File` class.
- Create the file using the `w` flag to write on it.
- The file has the extension `csv`.
- Use the current time on Unix to form the file name.
- Save the file in the local file system.

All of this is in one little sentence. A ruby programmer probably knows enough about the `File` and `Time` classes to understand the entire sentence without asking for the `w` flag or the `to_i` method.

But what is sure is that it doesn't want to know all these things at this point. Maybe it just wants to add a user field to the result or change the query to filter users.

The reader doesn't care what class you are using to create the file. Or where you are saving it, nor the name of the file. Are you using the `File` class or a wrapper to handle files? Are you saving the file to the local system or in a document-based database? It does not care.

When it needs to know about the file, it can go to the implementation details and read carefully how it is created and where it is placed. In the meantime, we don't want to fill its head with trivial details.

```ruby
class UsersExporter

  # more code

  def export
    users = find_users_to_export

    file = create_destination_file

    write_users(file, users)
  end

  # more

end
```

Now the reader can know what the code does by reading just three simple sentences and can choose what to read next.

## A different example

It's not only a backend thing, we can apply this recommendation to any other language or framework.

Let's look at a quick example of a useful single-line method in Vue:

```js
export default {

  // more stuff...

  async created() {
    this.users = await UsersService.index(this.params);
  }
}
```

Would it seem silly to create a single-line method that wraps the content of the `created` method? Maybe... But what if we need to get the users after a different event? Do we want to duplicate the method?

```js
export default {

  // more stuff...

  async created() {
    this.users = await UsersService.index(this.params);
  }

  methods() {
    onFilter() {
      this.users = await UsersService.index(this.params);
    }
  }
}
```

If we extract the method, we are improving our code. The top-level method is more abstract: the reader knows that we are getting the users when the component is created, but it does not know how and it does not care at this point. And, we are reusing code: if we need to change the `UsersService.index` signature, we need to modify only one line, not two or more calls in each file.

```js
export default {

  // more stuff...

  async created() {
    this.getUsers();
  }

  methods() {
    onFilter() {
      this.getUsers();
    },
    getUsers() {
      this.users = await UsersService.index(this.params);
    }
  }
}
```

## Conclusion

That is all. These might seem like minor details, but this care about minor details is what keeps our code clean and prevent us from losing control of it. Keep your readers focused on what matters at the moment. I hope you have enjoyed it!
