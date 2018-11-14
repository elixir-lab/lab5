# Make the Game a Service

Our Game module is currently just library code. It runs in the process
that calls it, and that calling process must keep the game's state,
passing it back with each call.

We're going to change this, making the Game into a server.

Rather than having `new_game` return a struct containing the state, it
will now return a pid. That pid will be passed to `make_move` and
`tally` in the same way the state was previously.

From the client's perspective, there is only one change to the API.
Whereas `make_move` used to return `{ game, tally }`, it will now return
just the tally: the state is held internally in the process, and the pid
never changes.

By the end of this process, you'll be able to do:

    iex> game = Hangman.new_game
    iex> tally = Hangman.make_move(game, "a")
    iex> tally = Hangman.make_move(game, "s")

The tests and the human player code have been updated to use this
slightly different API.

Remember to split your code into an implementation, server, and API.
(The API part is `lib/hangman.ex`. You just need to change `defdelegate`
into GenServer calls).

Also, I'm adding a new API function, `new_word/1`, which lets us force a
particular word to be used.

It'll end up looking something like this:


~~~ elixir
defmodule Hangman do

  def new_game() do
    new_game(Dictionary.random_word)
  end

  def new_game(word) do
    { :ok, pid } = GenServer.start_link(Hangman.Server, word)
    pid
  end

  def make_move(pid, guess) do
    GenServer.call(pid, { :make_move, guess })
  end

  def tally(pid) do
    GenServer.call(pid, { :tally })
  end

end
~~~