# Turn Tracker
> Hello! Welcome to my turn tracking project.
> Originally, I was making a game system and had to add a turn-tracking mechanic.
> Well, after finishing the logic, I realized that this potentially could be extended and used within other projects.
> And so, I published this turn tracker to npm.
>
> I figure it can support more than just game mechanics.
> For example, you could use a system like this to track who is supposed to do chores next, or something like that.
> It can support any system that needs to handle round-robin style handling, and needs to know "what is the next turn".

## How to Install
You can install the npm package by running `yarn add @noltron000/turn-tracker` or `npm install @noltron000/turn-tracker`.

## Navigating the Project's Exports
There are a few exports in this project, including some classes and helper types.
The classes are probably more important to document, so I'll highlight them here.

**The two main exports are:**
- the `TurnTracker` class
- the `TurnTrackerEvents` class

The `TurnTracker` class is stateful queue manager.
It keeps the list of turns, tracks the current turn, and advances through the roster as turns are added, removed, or skipped.

The `TurnTrackerEvents` class is a helper for listening to lifecycle changes, such as:
- when the tracker activates or deactivates
- when a round starts or ends
- when turns are added, removed, started, skipped, or ended

### `TurnTracker` versus `TurnTrackerEvents`
`TurnTracker` owns the list and the current state. The `TurnTrackerEvents` class dispatches callbacks for important state changes.
You will only be using events if you need to hook in custom functionality between turns and rounds and such.

### The Basics Using `TurnTracker`
```ts
import {TurnTracker} from "turn-tracker"

const tracker = new TurnTracker({
	items: ['Alice', 'Bob', 'Charlie'],
})

tracker.activate( )
console.log(tracker.currentTurn) // 'Alice'

tracker.next( )
console.log(tracker.currentTurn) // 'Bob'

tracker.add('Dana')
console.log(tracker.turns) // ['Alice', 'Bob', 'Charlie', 'Dana']
```

The tracker maintains ordering and current position in a round-robin style.
That means it can cycle through turns without losing track of which participant is currently active.
It also does not mutate the list between each turn, unlike some turn trackers.
Some turn tracker rely on looking at index-zero to determine the current turn.

### Working with Events
```ts
import {TurnTracker} from "turn-tracker"

const tracker = new TurnTracker({
	items: ['A', 'B', 'C'],
	events: {
		'start-turn': (tracker, turn) => {
			console.log(`Now playing: ${turn}`)
		},
		'end-turn': (tracker, turn) => {
			console.log(`Finished: ${turn}`)
		},
	},
})

tracker.activate( )
tracker.next( )
```

This emits events for turn and round lifecycle transitions, including the ability to respond to additions and removals as the tracker changes.

### Catalogue of `TurnTracker` Properties & Methods
#### Properties
- **`turns`**
	- A readonly snapshot of the current turn list.
- **`active`**
	- Whether the tracker is currently active and has a selected turn.
- **`currentIndex`**
	- The index of the active turn in the list, or `null` if inactive.
- **`currentTurn`**
	- The active turn value, or `null` if inactive.
- **`turnCounter`**
	- The number of turns that have progressed.
- **`roundCounter`**
	- The number of rounds that have completed.
- **`turnOrdinal`**
	- The numeric representation of the current turn (`turnCounter + 1`).
- **`roundOrdinal`**
	- The numeric representation of the current round (`roundCounter + 1`).

#### Turn Management Methods
- **`add(...turns)`**
	- Adds one or more turns to the tracker while preserving sort order and maintaining the current index.
- **`remove(...turns)`**
	- Removes one or more turns from the tracker while updating the current turn if needed.
- **`sort()`**
	- Re-sorts the internal list and re-syncs the current turn position.
- **`getEntryAt(index)`**
	- Gets the turn at a specific numeric index or relative turn position such as `'next-turn'`.

#### Tracker State Methods
- **`activate()`**
	- Activates the tracker and selects the first valid turn.
- **`deactivate({skip})`**
	- Deactivates the tracker and ends the current turn.
	- By default, the currently active turn is marked as skipped.
		Change `skip` if you would like to switch this behavior.
- **`next({skip})`**
	- Advances to the next valid turn.
	- By default, the currently active turn is marked as completed, not skipped.
		Change `skip` if you would like to switch this behavior.

#### Metadata & Serialization Methods
- **`toString()`**
	- Produces a readable summary of the current tracker state and the turn list.
- **`toData()`**
	- Returns a plain object containing the tracked turns, current index, and counters.
- **`toJSON()`**
	- Serializes the tracker state to JSON.
- **`toJSON5()`**
	- Serializes the tracker state to JSON5.

### Supported Event Names
The tracker emits a curated set of event names that can be subscribed to.

#### State events
- **`activate-tracker`**
- **`deactivate-tracker`**
- **`start-round`**
- **`end-round`**

#### Turn events
- **`add-turn`**
- **`remove-turn`**
- **`start-turn`**
- **`skip-turn`**
- **`end-turn`**

These events can help hook into gameplay flow, UI updates, or any other logic that depends on turn progression.

### Custom Sorting & Filtering
A `TurnTracker` can be initialized with custom sorting and filtering logic.

```ts
const tracker = new TurnTracker({
	items: [{name: 'A', priority: 2}, {name: 'B', priority: 1}],
	sortMethod: (a, b) => b.priority - a.priority,
	filterMethod: (turn) => turn.priority > 0,
})
```

This allows the tracker to order turns based on priority and ignore entries that should not currently participate in the cycle.

### Why This Exists
Compared to basic solutions, this turn tracker is meant to answer common questions when dealing with turns and round-robin systems, including:
- Which turn is active right now?
- What is the next valid turn?
- When a turn is removed or skipped, what should happen next?
- Should round counters increment when the cycle wraps?
- Is the turn order currently "offline" or "deactivated"?

This project addresses those concerns in a reusable, object-oriented way, while still remaining lightweight enough to integrate into games or other scheduling logic.
