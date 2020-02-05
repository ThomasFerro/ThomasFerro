# DDD in action: Armadora - The board game

> Applying DDD tactical patterns to the making of a game.

How do you apply most of the DDD tools when you have no domain experts nor any team at all?

Since I want to train using those tools whenever possible, I decided to build what seems to be a good experience for me: adapting a board game.

The domain rules are simply the game's ones, and discussions with domain experts can be reduced to reading the guide!

However, I can only apply tactical patterns, which is a simplistic take on DDD. The discovery of the domain and the exchanges with domain experts are a critical component of DDD and ignoring it in a real project is nonsensical to me. 

Anyway, let me introduce you to the game that I am building! 

## Game's rules

![Game in progress](https://github.com/ThomasFerro/readmes/blob/master/posts/12-ddd-in-action-armadora-the-board-game/game_in_progress.png)

Here are the rules, based on the guide. The wording is important since it is already the Ubiquitous Language. No need to work with domain experts to build it in our case. Note that I will be only implementing the basic rules, not the advanced ones.

Armadora is a game where every **player** will try to get their hands on the most **gold** possible.

![Board](https://github.com/ThomasFerro/readmes/blob/master/posts/12-ddd-in-action-armadora-the-board-game/board.png)

The **game board** is a 8x5 grid with two types of **cell**: the **lands** where the players will be able to put their **warriors** (more on that later) and the **gold stacks** with various quantity or gold.

The grid can be divided into **territories** by putting **palisades** between cells. A territory *must be at least four squares wide*.

At the beginning of a game, each player chooses a character (*Orc*, *Goblin*, *Elf* or *Mage*) and hide their warriors behind their screens.

Each player starts with the same army, depending on the number of players:

- For two players, each one get *11 warriors of 1 point*, *2 warriors of 2 points*, *1 warrior of 3 points*, *1 warrior of 4 points* and *1 warrior of 5 points*;
- For three players, each one get *7 warriors of 1 point*, *2 warriors of 2 points*, *1 warrior of 3 points* and *1 warrior of 4 points*;
- For four players, each one get *5 warriors of 1 point*, *1 warrior of 2 points*, *1 warrior of 3 points* and *1 warrior of 4 points*;

In the advanced rules set, the players also have power token based on their race and a reinforcement token.

*Forty gold* are then distributed randomly in eight piles: 1 pile of 3, 2 piles of 4, 2 piles of 5, 2 piles of 6 and 1 pile of 7.

When it is his turn, the player have to choose one of the following actions:

- Put one of a remaining warrior tile on an empty cell, with the number hidden;
- Put **one or two palisades** on the board, in an authorized border of a cell (one cannot put a palisade if it closes a territory of less than four cells);
- Use the race's power (only in advanced rules);
- Use reinforcement (only in advanced rules);
- Pass his turn.

Once a player passed his turn, he cannot play anymore for the rest of the game.

The game ends once every player has passed they turn. Every armies' strength is revealed and the gold of each territory is given to the player with the greatest army.

![End of a game](https://github.com/ThomasFerro/readmes/blob/master/posts/12-ddd-in-action-armadora-the-board-game/end_of_a_game.png)

In case of a tie, the players will compare their piles of gold, from highest to lowest.

Example: If the Elf have a pile of 6, a pile of 4 and a pile of 3 and the Orc have a pile of 6, one of 5 and one of 2, the Orc wins.

In a four-player game, the facing players can play as partners. We will not be implementing this feature in the first version.

## Building the game

Now that the *what* is clear, we can focus on the *how*. 

The game is implemented as a Go application, serving the information to the clients via an HTTP API and Web Sockets. These choices can and will be challenged later, but the first thing to focus on is the domain.

Every piece of code regarding the domain is written following the **Test Driven Development** principles. This way, I will be sure that every vital piece of the game is tested and functional.

The infrastructure matters are not tested at first, which I regret, but it was the most challenging part for me to do. Being a Go and especially a Web Sockets newbie, writing testable code for this part felt unnatural and I wanted to focus on delivering the value. However, as we will see in the last section, all of this layer will be re-written to be easily deployable.

On a brighter side, let us talk architecture. As the project is turn-based, it felt natural for me to build the game as an *event driven* system. We have a *Game* aggragate, on which domain events are applied to modify its state. We will see examples of events throughout the post. The main idea is to persist the way that lead to the current state of the system instead of the state itself.

The *events*, *commands*, *aggregates* and *structures* are based on the guide, which provides the *ubiquitous language*.

I choose not to apply *CQRS* patterns for now since there is no need to decorrelate the *command* and the *query* models. Again, this choice can be challenged.

Before diving into the game's implementation, let me first introduce the main concepts that we will see through the article.

### Concepts

**Domain events** help us understand what happened in the system. They contain information about what happened via their type and their inner data.

I decided to create non-technical *error events* in order to keep a log of unwanted things happening. In practice, this means that an invalid move will produce a domain event.

The main part of the domain is the **aggregate**. There is only one here, the *Game*, on which domain events will be applied in order to modify his state.

**Interactions** with the domain and more specifically the aggregate are achieved using **commands**.

For instance, a player will make a move via a command. This command will receive the *history of domain events* for the game and a *payload*. If needed, an aggregate can be built using the history and decisions can be made using the game's state. In the end, the command will return **the new domain events** that occurred on the aggregate.

```go
func SampleCommand(history []event.Event, commandPayload CommandPayload) []event.GameCreated {
    currentStateOfTheGame := game.ReplayHistory(history)
    // [...] Decisions based on the current state of the game.
    return []event.Event{
        event.AnEvent{
            // Payload
        },
        event.AnotherEvent{},
    }
}
```

Those concepts and more are also introduced in my [feedback on the four days DDD training that I followed last year](https://dev.to/thomasferro/summary-of-a-four-days-ddd-training-5a3c).

I will now briefly iterate through the different parts of the domain, explaining how I implemented them.

### Initialization of a game

This first part will introduce many of the building blocks of the application, while implementing the setup of a game.

The initial command is the one used to create a game, the aptly named `CreateGame`. No big deal here, just a method that returns a `GameCreated` event.

```go
func CreateGame() event.GameCreated {
	return event.GameCreated{}
}
```

We need a mechanism to be able to create a *Game* aggregate based on a history of events. Using a *loop* and a *type switch*, we will route the event to the right method on the aggregate.

```go
func ReplayHistory(history []event.Event) Game {
	var returnedGame Game
	returnedGame = game{}
	for _, nextEvent := range history {
		switch typedEvent := nextEvent.(type) {
		case event.GameCreated:
            returnedGame = returnedGame.ApplyGameCreated(typedEvent)
        // [...] One case per managed event type
		}
	}
	return returnedGame
}
```

The `ApplyGameCreated` simply changes the game's state, making it waiting for players to join.

```go
func (g game) ApplyGameCreated(event event.GameCreated) Game {
	g.state = WaitingForPlayers
	return g
}
```

Note that every `Apply` method return a new aggregate, to avoid side-effect.

Now that we have a game, we need to provide a way for players to join it. Another command will do the trick: `JoinGame`. This command, contrary to the previous one, needs a little bit of context provided by the payload.

```go
type JoinGamePayload struct {
	Nickname  string
	Character character.Character
}
```

For the command to be fulfilled, the game needs to be in the right state, the game must not be already full and the character must not have been already taken:

```go
func JoinGame(history []event.Event, joinGamePayload JoinGamePayload) []event.Event {
	gameToJoin := game.ReplayHistory(history)

	if gameToJoin.State() != game.WaitingForPlayers {
		return []event.Event{
			event.GameAlreadyStarted{},
		}
	}

	if len(gameToJoin.Players()) == 4 {
		return []event.Event{
			event.GameAlreadyFull{},
		}
	}

	for _, player := range gameToJoin.Players() {
		if player.Character() == joinGamePayload.Character {
			return []event.Event{
				event.CharacterAlreadyChosen{
					Character: joinGamePayload.Character,
				},
			}
		}
	}

	return []event.Event{
		event.PlayerJoined{
			Nickname:  joinGamePayload.Nickname,
			Character: joinGamePayload.Character,
		},
	}
}
```

You have here examples of error events. I could have decided to implement them as returned error, but it felt more declarative this way. Another implementation choice that will certainly be challenged later.

Once two to four players joined the game, it can be started via the *`StartTheGame` command*.

The first thing to do is to check if there is enough players:

```go
func StartTheGame(history []event.Event) []event.Event {
	newGame := game.ReplayHistory(history)

	if len(newGame.Players()) < 2 {
		return []event.Event{
			event.NotEnoughPlayers{
				NumberOfPlayers: len(newGame.Players()),
			},
		}
    }

    // [...]
```

When the game starts, the gold, palisades and warriors are distributed. How to represent that? With domain events!

The `StartTheGame` command can be completed with those events as following:

```go
func StartTheGame(history []event.Event) []event.Event {
	newGame := game.ReplayHistory(history)

	if len(newGame.Players()) < 2 {
		return []event.Event{
			event.NotEnoughPlayers{
				NumberOfPlayers: len(newGame.Players()),
			},
		}
	}

	events := []event.Event{}

	warriorsToDistribute := warrior.WarriorsToDistribute(len(newGame.Players()))

	events = append(events, event.WarriorsDistributed{
		WarriorsDistributed: warriorsToDistribute,
	})

	goldStacksToDistribute := gold.GoldToDistribute()

	events = append(events, event.GoldStacksDistributed{
		GoldStacks: goldStacksToDistribute,
	})

	events = append(events, event.PalisadesDistributed{
		Count: 35,
	})

	return append(events, event.GameStarted{})
}
```

The `WarriorsToDistribute` and `GoldToDistribute` methods are extracted in another package for the sake of readability and because it is out of the command's responsibility.

Those events, when applied in the aggregate, put the game in the right state:

```go
func (g game) ApplyGameStarted(event event.GameStarted) Game {
	g.state = Started
	return g
}

func (g game) ApplyWarriorsDistributed(event event.WarriorsDistributed) Game {
	players := []Player{}
	for _, player := range g.Players() {
		players = append(players, player.SetWarriors(event.WarriorsDistributed))
	}
	g.players = players
	return g
}

func (g game) ApplyGoldStacksDistributed(event event.GoldStacksDistributed) Game {
	g.board = board.NewBoard(event.GoldStacks)
	return g
}

func (g game) ApplyPalisadesDistributed(event event.PalisadesDistributed) Game {
	g.board = g.Board().SetPalisadesLeft(event.Count)
	return g
}
```

I made the `player`'s `SetWarriors` and the `board`'s `SetPalisadesLeft` methods to return a new instance that is why I reassign them to the game before returning it.

Now that the game is started, it is time for the players to actually play!

### Anatomy of a turn

In standard rules, three actions are available: put a warrior on a land, put one or two palisades between cells or pass the turn.

Those three commands have two things in common. They can only be done by the current player, which is checked with the following assertion:

```go
	currentGame := game.ReplayHistory(history)

	if currentGame.CurrentPlayer() != commandPayload.Player {
		return []event.Event{
			event.NotThePlayerTurn{
				PlayerWhoTriedToPlay: commandPayload.Player,
			},
		}
    }
    // [...]
```

They also all share the fact that the turn of the player ends after the command is handled. This is represented by the `NextPlayer` event.

Going to the more specific stuff, the `PutWarrior` command must check that the player have warrior of the requested strength left and that the cell is an empty land.

```go
func PutWarrior(history []event.Event, payload PutWarriorPayload) []event.Event {
	currentGame := game.ReplayHistory(history)

	if currentGame.CurrentPlayer() != payload.Player {
		return []event.Event{
			event.NotThePlayerTurn{
				PlayerWhoTriedToPlay: payload.Player,
			},
		}
	}

	currentPlayer := currentGame.Players()[currentGame.CurrentPlayer()]

	if getWarriorsLeft(currentPlayer.Warriors(), payload.Warrior) == 0 {
		return []event.Event{
			event.NoMoreWarriorOfThisStrength{
				Strength: payload.Warrior,
			},
		}
	}

	if cellAlreadyTaken(currentGame.Board(), payload.Position) {
		return []event.Event{
			event.CellAlreadyTaken{
				Position: payload.Position,
			},
		}
	}

	return []event.Event{
		event.WarriorPut{
			Player:   payload.Player,
			Strength: payload.Warrior,
			Position: payload.Position,
		},
		event.NextPlayer{},
	}
}
```

The PutPalisades is more complex since it must check the grid's validity after putting the requested palisades. If putting the palisades creates at least on territory of less than 4 cells, the move is invalid.

Those *territories* matters are extracted in an eponymous package, out of the DDD scope. It is not the piece of code that I am the most proud of, but you can find it on the [project's repository](https://github.com/ThomasFerro/armadora). *PR welcomed*, as they say!

I am not going into much details about passing a turn, the only notable thing is the *end game management*. The command needs to check if every player passed they turn, and ends the game if they all did via a *`GameFinished` event*. This event contains the final score, computed by a `territory` and a `score` services. As stated earlier, this is the most complex part of the application, domain insight might help me refactor it later.

### End of a game

We are almost done with the basic rules! The only missing part is the ability to end the game, and provide a scoreboard.

The guide indicate us that the game ends once every player have passed they turn. We can represent that with a `GameFinished` event. It has to be checked every time a player pass, so we can add that to the `PassTurn` command:

```go
func nextPlayerOrEndGame(history []event.Event) event.Event {
	currentGame := game.ReplayHistory(
		append(history),
	)
	for _, player := range currentGame.Players() {
		if !player.TurnPassed() {
			return event.NextPlayer{}
		}
	}

	territories, _ := board.FindTerritories(currentGame.Board())

	return event.GameFinished{
		Scores: score.ComputeScores(territories),
	}
}
```

Again, the complexity of score computing is extracted in its own package, along with ties management.

The `GameFinished` event is handled by the game by changing his state and adding the scores:

```go
func (g game) ApplyGameFinished(event event.GameFinished) Game {
	g.state = Finished
	g.scores = event.Scores
	return g
}
```

There we go, three pages of rules implemented following the nomenclature!

## What is next?

You may have noticed, this article is focused on the domain concerns. As the application was built with agility in mind, every part was delivered with the corresponding front-end client. This front-end is barebones and I still need to build a real responsive Web App with the game's assets before delivering the V1.

Another area of improvement is on the infrastructure part. The Go application is stateful, with the events and the parties stored in memory. These information will need to be extracted in any sort of repository and the application will have to fetch those data on every request. It will make the application Serverles-ready and more easy to deploy.

After that, a *continuous delivery* pipeline will have to be built to facilitate the first release and the following ones.

I think that there is also some work to be done on the client-server relationship. The client still have too many responsibilities in my opinion, and it would be great for the server to send him available moves for example. Using the right protocol for this turn-base game is also important to me, the persistent connection of the Web Sockets seems exaggerated.

Advances rules are still to be implemented, such as the team-play for four players or the integration of reinforcement and race's powers.

## What to get out of this experience

It was a really great experience for me! Building the game with *DDD* patterns and following *TDD* principles made the process feel more natural.

There is no translation from and to domain terms thanks to the *ubiquitous language*. The testing was very explicit thanks again to domain events. In the end, I found the domain source code to be more obvious and to the point.

I am still frustrated about the deployment part, which is still not done by the time I am writing, but it was out of the score of this whole experience.

To conclude, I cannot recommend you enough to read about DDD and try it out yourself!
