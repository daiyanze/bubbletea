Bubble Tea Basics
=================

Bubble Tea is based on the functional design paradigms of [The Elm
Architecture][elm] which happens work nicely with Go. It's a delightful way to
build applications.

By the way, the non-annotated source code for this program is available
[on GitHub](https://github.com/charmbracelet/bubbletea/tree/master/tutorials/basics).

This tutorial assumes you have a working knowledge of Go.

[elm]: https://guide.elm-lang.org/architecture/

## Enough! Let's get to it.

For this tutorial we're making a to-do list.

To start we'll define our package and import some libraries. Our only external
import will be the Bubble Tea library, which we'll call `tea` for short.

```go
package main

import (
    "fmt"
    "os"

    tea "github.com/charmbracelet/bubbletea"
)
```

Bubble Tea programs are comprised of a **model** that describes the application
state and three simple methods on that model:

* **Init**, a function that returns an initial command for the application to run.
* **Update**, a function that handles incoming events and updates the model accordingly.
* **View**, a function that renders the UI based on the data in the model.

## The Model

So let's start by defining our model which will store our application's state.
It can be any type, but a `struct` usually makes the most sense.

```go
type model struct {
    choices  []string           // items on the to-do list
    cursor   int                // which to-do list item our cursor is pointing at
    selected map[int]struct{}   // which to-do items are selected
}
```

## Initialization

Next we'll define our application’s initial state. We’ll store our initial
model in a simple variable, and then define the `Init` method.  `Init` can
return a `Cmd` that could perform some initial I/O. For now, we don't need to
do any I/O, so for the command we'll just return `nil`, which translates to "no
command."

```go
var initialModel = model{
    // Our to-do list is just a grocery list
    choices:  []string{"Buy carrots", "Buy celery", "Buy kohlrabi"},

    // A map which indicates which choices are selected. We're using
    // the  map like a mathematical set. The keys refer to the indexes
    // of the `choices` slice, above.
    selected: make(map[int]struct{}),
}

func (m model) Init() tea.Cmd {
    // Just return `nil`, which means "no I/O right now, please."
    return nil
}
```

## The Update Method

Next we'll define the update method. The update function is called when
"things happen." Its job is to look at what has happened and return an updated
model in response to whatever happened. It can also return a `Cmd` and make
more things happen, but for now don't worry about that part.

In our case, when a user presses the down arrow, `update`'s job is to notice
that the down arrow was pressed and move the cursor accordingly (or not).

The "something happened" comes in the form of a `Msg`, which can be any type.
Messages are the result of some I/O that took place, such as a keypress, timer
tick, or a response from a server.

We usually figure out which type of `Msg` we received with a type switch, but
you could also use a type assertion.

For now, we'll just deal with `tea.KeyMsg` messages, which are automatically
sent to the update function when keys are pressed.

```go
func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {

    // Is it a key press?
    case tea.KeyMsg:

        // Cool, what was the actual key pressed?
        switch msg.String() {

        // These keys should exit the program.
        case "ctrl+c", "q":
            return m, tea.Quit

        // The "up" and "k" keys move the cursor up
        case "up", "k":
            if m.cursor > 0 {
                m.cursor--
            }

        // The "down" and "j" keys move the cursor down
        case "down", "j":
            if m.cursor < len(m.choices)-1 {
                m.cursor++
            }

        // The "enter" key and the spacebar (a literal space) toggle
        // the selected state for the item that the cursor is pointing at.
        case "enter", " ":
            _, ok := m.selected[m.cursor]
            if ok {
                delete(m.selected, m.cursor)
            } else {
                m.selected[m.cursor] = struct{}{}
            }
        }
    }

    // Return the updated model to the Bubble Tea runtime for processing.
    // Note that we're not returning a command.
    return m, nil
}
```

You may have noticed that "ctrl+c" and "q" above return a `tea.Quit` command
with the model. That's a special command which instructs the Bubble Tea runtime
to quit, exiting the program.

## The View Method

At last, it's time to render our UI. Of all the methods, the view is the
simplest. We look at the  model in it's current state and use it to return
a `string`.  That string is our UI!

Because the view describes the entire UI of your application, you don't have
to worry about redraw logic and stuff like that. Bubble Tea takes care of it
for you.

```go
func (m model) View() string {
    // The header
    s := "What should we buy at the market?\n\n"

    // Iterate over our choices
    for i, choice := range m.choices {

        // Is the cursor pointing at this choice?
        cursor := " " // no cursor
        if m.cursor == i {
            cursor = ">" // cursor!
        }

        // Is this choice selected?
        checked := " " // not selected
        if _, ok := m.selected[i]; ok {
            checked = "x" // selected!
        }

        // Render the row
        s += fmt.Sprintf("%s [%s] %s\n", cursor, checked, choice)
    }

    // The footer
    s += "\nPress q to quit.\n"

    // Send the UI for rendering
    return s
}
```

## All Together Now

The last step is to simply run our program. We pass our initial model to
`tea.NewProgram` and let it rip:

```go
func main() {
    p := tea.NewProgram(initialModel)
    if err := p.Start(); err != nil {
        fmt.Printf("Alas, there's been an error: %v", err)
        os.Exit(1)
    }
}
```

## What's Next?

This tutorial covers the basics of building an interactive terminal UI, but
in the real world you'll also need to perform I/O. To learn about that have a
look at the [Command Tutorial][cmd]. It's pretty simple.

There are also several [Bubble Tea examples][examples] available and, of course,
there are [Go Docs][docs].

[cmd]: http://github.com/charmbracelet/bubbletea/tree/master/tutorials/commands/
[examples]: http://github.com/charmbracelet/bubbletea/tree/master/examples
[docs]: https://pkg.go.dev/github.com/charmbracelet/bubbletea?tab=doc

### Bubble Tea in the Wild

For some Bubble Tea programs in production, see:

* [Glow](https://github.com/charmbracelet/glow): a markdown reader, browser and online markdown stash
* [The Charm Tool](https://github.com/charmbracelet/charm): the Charm user account manager

### Libraries we use with Bubble Tea

* [Bubbles][bubbles]: various Bubble Tea components
* [Termenv][termenv]: Advanced ANSI styling for terminal applications
* [Reflow][reflow]: ANSI-aware methods for formatting and generally working with text. Of particular note is `PrintableRuneWidth` in the `ansi` sub-package which measures the physical widths of strings. Many runes, such as East Asian characters, emojis, and various unicode symbols are two cells wide, so measuring a layout with `len()` often won't cut it. Reflow is particularly nice for this as it measures character widths while ignoring any ANSI sequences present.

[termenv]: https://github.com/muesli/termenv
[reflow]: https://github.com/muesli/reflow
[bubbles]: https://github.com/charmbracelet/bubbles

### Feedback

We'd love to hear your thoughts on this tutorial. Feel free to drop us a note!

* [Twitter](https://twitter.com/charmcli)
* [The Fediverse](https://mastodon.technology/@charm)
