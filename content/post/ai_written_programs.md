+++
title = 'AI Written Programs'
date = 2024-08-05T19:27:31-05:00
draft = false
tags = ["ai", "prompt_engineering", "python", "claude", "chatgpt"]
summary = "What does it look like when you attempt to write a whole program with AI?"
+++
# Guiding AI / AI Guiding You Through a Whole Program

In what will surely not be my last reference to my friend Brandon's work, he recently posted about [A Small but Complete AI-Written Program](https://bcorfman.github.io/post/tic_tac_toe_board_states/). What's particularly great about his article is that you get to see his prompts, some additional comments showing his thought process and then Claude's (in this case) responses. Again, for clarity, the purpose of the program is to generate a list of all valid tic-tac-toe game board states.

## A Quick Rerun of the Claude Experiment

I decided to follow along and see how closely my own attempt came using the exact same prompts. As expected, the responses are different because LLMs are usually non-deterministic in results due to the nature of the algorithms and techniques used. In Brandon's example, directly using Claude using the web UI, we're not able to control the "temperature" settings which also increases the entropy for results. But even when using an LLM with control to set *temperature* to 0, [there is non-determinism](https://152334h.github.io/blog/non-determinism-in-gpt-4/).

In my run at the same prompts, Claude gave an initial response that did include empty as a valid state, but got the wrong answer because final winning states were not included.

Here is Claude's first attempt for me at figuring out all valid states of tic-tac-toe:

```python
from itertools import product

def generate_tic_tac_toe_states():
    empty = '.'
    players = ['X', 'O']
    board = [empty] * 9
    states = set()

    def is_winner(board, player):
        winning_combinations = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8],  # Rows
            [0, 3, 6], [1, 4, 7], [2, 5, 8],  # Columns
            [0, 4, 8], [2, 4, 6]  # Diagonals
        ]
        return any(all(board[i] == player for i in combo) for combo in winning_combinations)

    def generate_states(board, player_index):
        board_tuple = tuple(board)
        if board_tuple in states or is_winner(board, players[1 - player_index]):
            return

        states.add(board_tuple)

        for i in range(9):
            if board[i] == empty:
                new_board = list(board)
                new_board[i] = players[player_index]
                generate_states(new_board, 1 - player_index)

    generate_states(board, 0)
    return states

states = generate_tic_tac_toe_states()
print(f"Number of tic-tac-toe game states: {len(states)}")
```

You can see, this approach is a bit a more recursive, calling the inner `generate_states` function recursively. The result is tidier than in Brandon's example, but a bit harder to reason about (in my personal opinion). Because I knew what the answer was, I knew that this program was incorrect, but it did take me a bit of head scratching to see the problem. Ultimately, *my* fix was to add the `states.add(board_tuple)` line above the check for prior state or `is_winner`. But the goal here is not have me code the solution, so I now had to deviate from Brandon's prompts. I could have played really dumb and told Claude that it was wrong, but I decided to point out more information about *what* was wrong.

My follow-up prompt:

> I believe there is a problem, you need to count winning states as a valid state as well.

And Claude's new program:

```python
from itertools import product

def generate_tic_tac_toe_states():
    empty = '.'
    players = ['X', 'O']
    board = [empty] * 9
    states = set()

    def is_winner(board, player):
        winning_combinations = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8],  # Rows
            [0, 3, 6], [1, 4, 7], [2, 5, 8],  # Columns
            [0, 4, 8], [2, 4, 6]  # Diagonals
        ]
        return any(all(board[i] == player for i in combo) for combo in winning_combinations)

    def generate_states(board, player_index):
        board_tuple = tuple(board)
        if board_tuple in states:
            return

        states.add(board_tuple)

        # Check if the game is over
        if is_winner(board, players[0]) or is_winner(board, players[1]) or '.' not in board:
            return

        for i in range(9):
            if board[i] == empty:
                new_board = list(board)
                new_board[i] = players[player_index]
                generate_states(new_board, 1 - player_index)

    generate_states(board, 0)
    return states

states = generate_tic_tac_toe_states()
print(f"Number of tic-tac-toe game states: {len(states)}")
```

This version does work correctly. Interestingly, it switched to checking for either player being a winner, whereas technically, it *could* just check for the prior player being a winner plus the cat state.

Of note, Claude has included the empty board state now in the count. Unlike Brandon, I didn't end up having it add that, but yet Claude thinks the empty board is still excluded, even though it's not. This ended up being an issue I had to deal with when I had it generate test cases.

I won't iterate over all the issues with tests, but again, like Brandon, it started with `unittest` instead of `pytest`. Very understandable, though. The tests were broken in many ways, some similar to what Brandon had encountered. Again, I had to point out flaws in logic and correct Claude on the empty state being included in the count and to use a precise number instead of a range for the valid result of the count.

In the end, I was able to end up with a complete program with tests, but it was a bit of a struggle. But setting aside the pain points, I still feel like starting with AI, you can still get a quick start on something and then take it the rest of the way yourself.

Thinking back now, I should have immediately just started with ChatGPT as a better comparison.

## A Quick ChatGPT Experiment

So as a quick experiment, I did a single run with ChatGPT as well. The results were similar but definitely different. The resulting program (without tests) was achieved with fewer issues:

```python
def is_valid_state(board):
    """
    Check if the given board is a valid state in tic-tac-toe.
    """
    x_count = sum(row.count('X') for row in board)
    o_count = sum(row.count('O') for row in board)

    if x_count not in {o_count, o_count + 1}:
        return False

    def is_winner(player):
        # Check rows, columns, and diagonals
        for i in range(3):
            if all(board[i][j] == player for j in range(3)):
                return True
            if all(board[j][i] == player for j in range(3)):
                return True
        if all(board[i][i] == player for i in range(3)):
            return True
        if all(board[i][2 - i] == player for i in range(3)):
            return True
        return False

    x_win = is_winner('X')
    o_win = is_winner('O')

    if x_win and o_win:
        return False
    if x_win and x_count == o_count:
        return False
    if o_win and x_count > o_count:
        return False

    return True

def generate_states(board, is_x_turn, unique_states):
    """
    Generate all possible game states from the current board.
    """
    # Convert board to a tuple of tuples to use as a dictionary key
    board_tuple = tuple(tuple(row) for row in board)
    if board_tuple in unique_states:
        return

    # Add the board to unique states if it's valid
    if is_valid_state(board):
        unique_states.add(board_tuple)
    else:
        return

    # Generate next states
    for i in range(3):
        for j in range(3):
            if board[i][j] == '':
                board[i][j] = 'X' if is_x_turn else 'O'
                generate_states(board, not is_x_turn, unique_states)
                board[i][j] = ''

def count_tic_tac_toe_states():
    """
    Count all unique tic-tac-toe game states, including the empty board.
    """
    initial_board = [['' for _ in range(3)] for _ in range(3)]
    unique_states = set()
    generate_states(initial_board, True, unique_states)
    # Add the empty board state manually
    unique_states.add(tuple(tuple(row) for row in initial_board))
    return len(unique_states)

# Count the total number of unique states
total_states = count_tic_tac_toe_states()
print(f"Total number of unique tic-tac-toe game states (including empty board): {total_states}")
```

The board in this version is represented as nested tuples and it may be a little easier to visualize and reason about in that regard. There's a bit less "trickeration" (my term) with turn-swapping a few other things. It kind of feels less math-y. The net result, though, is a program that's nearly twice as long as the working version from Claude, not that number of lines is everything, but it's worth mentioning.

Having ChatGPT write the tests, yielded similar problems (I'll save you from all the iterations). But despite me pointing out a flaw in winning states, ChatGPT never could seem to recover, instead generating more broken tests with an invalid winning state that was never going to be included in the set of valid end states. Here's where I ended my experiment:

```python
import pytest

# Assuming the functions from the original implementation are imported
# from tic_tac_toe import count_tic_tac_toe_states, generate_states

def test_total_number_of_states():
    # Known total number of unique states, including empty board
    # For a 3x3 tic-tac-toe board, there are 5478 possible game states (including the empty board).
    total_states = count_tic_tac_toe_states()
    assert total_states == 5478, "Total number of unique game states should be 5478"

def test_includes_empty_board():
    # Check that the empty board is included as a state
    empty_board = (('','',''), ('','',''), ('','',''))
    unique_states = set()
    generate_states([['' for _ in range(3)] for _ in range(3)], True, unique_states)
    assert empty_board in unique_states, "Empty board should be included in the states"

def test_includes_known_intermediate_state():
    # Check that a known intermediate state is included
    intermediate_state = (('X', 'O', 'X'), ('O', 'X', ''), ('', '', ''))
    unique_states = set()
    generate_states([['' for _ in range(3)] for _ in range(3)], True, unique_states)
    assert intermediate_state in unique_states, "Known intermediate state should be included in the states"

def test_includes_known_final_state():
    # Check that a known final state is included
    final_state = (('X', 'X', 'X'), ('O', 'O', ''), ('O', '', ''))
    unique_states = set()
    generate_states([['' for _ in range(3)] for _ in range(3)], True, unique_states)
    assert final_state in unique_states, "Known final state should be included in the states"

def test_does_not_include_invalid_state():
    # Check that an invalid state (e.g., two winners or wrong turn) is not included
    invalid_state = (('X', 'X', 'X'), ('O', 'O', 'O'), ('X', 'X', 'O'))
    unique_states = set()
    generate_states([['' for _ in range(3)] for _ in range(3)], True, unique_states)
    assert invalid_state not in unique_states, "Invalid state should not be included in the states"
```

## Conclusion

So I guess, TL;DR *caveat emptor*.

A couple of points I remind myself when using AI for code generation:

1. The same prompt can result in different approaches. So it's probably good to try a couple times to see which approach feels more correct. Note here that experience as a developer is your guide. A non-developer is still not going to have good results with this as demonstrated by the experiments in what is essentially a pretty basic example. Furthermore, a non-developer is not going to be able to evaluate structure or approaches as being better or worse even when AI yields a working program.
2. Don't trust or assume that what's generated will work or that it's correct. I've had good success using AI for helping get an *idea* about how to approach a problem or get started on a solution, but as demonstrated, it's a bit more laborious getting to a complete program.

{{< callout important "Final Summary" >}}
AI is useful for coding but don't count on it being able to get you 100% done or being 100% right.
{{< /callout >}}
