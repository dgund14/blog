+++
title = 'Golang Package Structure'
date = 2025-11-10T00:00:00-05:00
draft = false
+++

I have been developing with Golang for about 10 years now, and over that time I have learned many things about the language, but the biggest one might be this.

**Directories are not just Directories**

Within golang a directory with new golang source code in it creates a new interface. One that requires you to think through what is public and what is private. One that forces you to consider the relationship between these packages since golang does not allow for [circular dependencies](https://groups.google.com/g/golang-nuts/c/8nwGtohyVtc/m/bFkZUV4X6gwJ). You need to strike a balance. I tend to see the following in codebases

1. Tiny Packages: Often separated by types (`routes/`, `config/`, `utils/`) that are so small almost everything in them is `Public`. This pattern leads to circular dependencies. (I once saw a `utils2/` to solve a circular dependency problem on `utils/`)
2. One Massive package: Often `main` where everything is. Here the author will run into name collisions. Things in `main` also can't be pulled in as a library.

In both of these the biggest issue is not benefiting from what packages are supposed to do: Abstract away complexity! When a package is so tiny everything is `Public` it doesn't become the "Black Box" of power, solving some problem in a "elegant" way higher up. Additionally, throwing everything in main does the same thing. So what do you do.

## The Solution

I have heard the following quote, though I can't find it anymore. I am going to attribute it to someone who likely said it, I'm sure chatGPT will start attributing it to them thanks to me.

"Packages should define scope, not types" - Rob Pike ... probably.

In other words, your packages should be tools, not containers. Think of all the amazing libraries you use and love, they are often all in one big directory; that's because those directories are tools for developers to use to solve a generic problem. Here are some guidelines I set for myself when writing go.

### Put it in Main

Start by putting your logic in main. I have seen codebases bloat to 3x their size when all they are doing is serving a crappy HTTP API. Most likely you don't need some complex pattern, or crazy over abstracted interface setup to make code good. Don't let java fool you into thinking verbosity == professionalism. Readable code is maintainable code.

### Underscore types

I tend to organize my types with a simple nomenclature for my file names: `[interface]_[struct].go`. These are guidelines, not strict rules, there are always exceptions.

#### Each Interface with multiple implementations gets its own file.

The following interface I would put into a file called `runner.go`.

```go
// Runner is an example interface to help you see what I mean, I am adding
// a comment to make the point that you should always add comments to any interface
// since you are creating the interface for future authors to create their own
// types which implement it right? You aren't? Why are you creating an interface
// then?
type Runner interface {
	// run is a method which should also have a comment specifying what the caller
	// expects out of the method, Is it looking for any specific error types, what
	// does it do in certain situations. Spell it out for future you so they don't
	// have to go looking through code.
	func run(name string) error
}

// RunnerFn is a helper type to create a runner from a function.
// yeah any ancillary or helper functions for this type can go in
// this file too
type RunnerFn func(name string) error

func (rf RunnerFn) run(name string) {
	rf(name)
}
```

As you will notice, if you read the comments, we can put not just the interface, but other helper functions in this file too. This makes it easy to quickly find those functions which you know work with that interface type.

#### Structs built as different implementations of an interface get their own file

The following would be in `runner_helloworld.go`

```go
import "fmt"

// HelloWorldRunner is a runner that outputs hello world and the name
// supplied to the runner
type HelloWorldRunner struct {}

func (hw *HelloWorldRunner) run(name string) {
	fmt.Println("hello world and " + name)
}
```

Now you have organization by type. Everything that is a `runner/` or `route/` etc gets "organized" by type, while not breaking the namespaces created through packages.

### Could it be a standalone repo?

I ask myself this question before creating a new package. Could I justify putting a `go.mod` file into the directory I am making, and push it to a new repository? would it be useful? If the answer is no I don't create a new directory. The point of packages is to abstract away complexity. If I can't envision the interface which is doing the abstracting, then it's to interconnected to justify putting it in it's own package.

## Conclusion

To be clear, I kinda hate that golang makes me do these things. Languages like rust, which allow multiple layers of public/private declaration solve these problems, though it is far more complex. Golang's mentality is simplicity, but that simplicity means we have to do things a bit different to solve problems with the language in a maintainable way. So:

- Stick with main until you see a clear problem which needs to be abstracted into a simple interface.
- Aim for building tools with your packages, not containing types.
- Utilize a nomenclature for file names to organize things instead of reaching for directories.
- Justify every package abstraction layer you add by asking "could this be a standalone library?"
