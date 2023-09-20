# Project: [Hog](hog.py)

#### To play the game of Hog, run this command line in your terminal:
> python3 hog_gui.py -f

## Description: 

> In Hog, two players alternate turns trying to be the first to end a turn with at least 100 total points. On each turn, the current player chooses some number of dice to roll, up to 10. That player's score for the turn is the sum of the dice outcomes. If any of the dice outcomes is a 1, the current player's score for the turn is 1.

## Implementation:

> The `roll_dice` function takes two arguments: a positive integer called `num_rolls` giving the number of dice to roll and a `dice` function. It either returns the number of points scored by rolling the dice that number of times in a turn or the sum of the outcomes or 1 if any of the dice is 1.

```python
def roll_dice(num_rolls, dice=six_sided):
    """Roll DICE for NUM_ROLLS times.  Return either the sum of the outcomes,
    or 1 if a 1 is rolled (Pig out). This calls DICE exactly NUM_ROLLS times.

    num_rolls:  The number of dice rolls that will be made; at least 1.
    dice:       A zero-argument function that returns an integer outcome.
    """
    # These assert statements ensure that num_rolls is a positive integer.
    assert type(num_rolls) == int, 'num_rolls must be an integer.'
    assert num_rolls > 0, 'Must roll at least once.'
    "*** YOUR CODE HERE ***"
    a = 0
    total = 0
    one = False

    while a < num_rolls:
        b = dice()
        if b == 1:
            one = True
        else:
            total += b
        a += 1

    if one:
        return 1
    return total
```

> The `take_turn` function will take three arguments: a positive integer `num_rolls` giving the number of dice to roll, a positive integer `opponent_score` that is less than 100 for the 0 dice rule and a `dice` function. It either returns the number of point from calling the `roll_dice` or the `max(opponent_score % 10, opponenet_score // 10) + 1`(0 dice rule).

```python
def take_turn(num_rolls, opponent_score, dice=six_sided):
    """Simulate a turn rolling NUM_ROLLS dice, which may be 0 (Free bacon).

    num_rolls:       The number of dice rolls that will be made.
    opponent_score:  The total score of the opponent.
    dice:            A function of no args that returns an integer outcome.
    """
    assert type(num_rolls) == int, 'num_rolls must be an integer.'
    assert num_rolls >= 0, 'Cannot roll a negative number of dice.'
    assert num_rolls <= 10, 'Cannot roll more than 10 dice.'
    assert opponent_score < 100, 'The game should be over.'
    "*** YOUR CODE HERE ***"
    if num_rolls == 0:
        a = opponent_score % 10   
        b = opponent_score // 10
        c = max(a, b) + 1
        return c
    else:
        return roll_dice(num_rolls, dice)
```

> The `select_dice` function take two arguments: two positive integer `score` and `opponent_score` which represent the score of two player. It returns a `four_sided` dice if the sum of two players' score are divisible by 7 else returns a `six_sided` dice.

```python
def select_dice(score, opponent_score):
    """Select six-sided dice unless the sum of SCORE and OPPONENT_SCORE is a
    multiple of 7, in which case select four-sided dice (Hog wild).

    >>> select_dice(4, 24) == four_sided
    True
    >>> select_dice(16, 64) == six_sided
    True
    >>> select_dice(0, 0) == four_sided
    True
    """
    "*** YOUR CODE HERE ***"
    if (score + opponent_score) % 7 == 0:
        return four_sided
    return six_sided
```

> The `play` function takes three arguments: two strategy function `strategy0` and `strtegy1` that will return the number of rolls for a turn giving the `score` and `opponent_score`, and positive integer `goal` that represents the winning score for the game(default set to 100). It will swap the current player's score with the opponent's score if the current score is half of the opponent's score or the opponent's score is half of the current player's score. It also simulates a game and return the final scores of both players when the game is over.

```python
def play(strategy0, strategy1, goal=GOAL_SCORE):
    """Simulate a game and return the final scores of both players, with
    Player 0's score first, and Player 1's score second.

    A strategy is a function that takes two total scores as arguments
    (the current player's score, and the opponent's score), and returns a
    number of dice that the current player will roll this turn.

    strategy0:  The strategy function for Player 0, who plays first.
    strategy1:  The strategy function for Player 1, who plays second.
    """
    who = 0  # Which player is about to take a turn, 0 (first) or 1 (second)
    score, opponent_score = 0, 0
    "*** YOUR CODE HERE ***"

    while score < goal or opponent_score < goal:
        score += take_turn(strategy0(score, opponent_score), opponent_score, dice=select_dice(score, opponent_score))
        
        if score >= 100:
            return score, opponent_score
        
        if opponent_score > 0 and score > 0 and (opponent_score / score == 2 or score / opponent_score == 2):
            score, opponent_score = opponent_score, score
        
        opponent_score += take_turn(strategy1(opponent_score, score), score, dice=select_dice(opponent_score, score))
        
        if opponent_score >= 100:
            return score, opponent_score
        
        if opponent_score > 0 and score > 0 and (opponent_score / score == 2 or score / opponent_score == 2):
            score, opponent_score = opponent_score, score
```

> The `make_averaged` function will take two arguments: `fn` is any function that return integer and a positive integer `num_samples` is the number of time that run the `fn`. It returns the average of the `fn` after it runs in `num_samples` times.

```python
def make_averaged(fn, num_samples=1000):
    """Return a function that returns the average_value of FN when called.

    To implement this function, you will have to use *args syntax, a new Python
    feature introduced in this project.  See the project description.

    >>> dice = make_test_dice(3, 1, 5, 6)
    >>> averaged_dice = make_averaged(dice, 1000)
    >>> averaged_dice()
    3.75
    >>> make_averaged(roll_dice, 1000)(2, dice)
    6.0

    In this last example, two different turn scenarios are averaged.
    - In the first, the player rolls a 3 then a 1, receiving a score of 1.
    - In the other, the player rolls a 5 and 6, scoring 11.
    Thus, the average value is 6.0.
    """
    "*** YOUR CODE HERE ***"
    def aver(*args):
        i, result = 0, 0
        while i < num_samples:
            result += fn(*args)
            i += 1
        return result / num_samples
    return aver
```

> The `bacon_strategy` will take two arguments:  two positive integer `score` and `opponent_score` which represent the score of two player. It returns the number of rolls base on the `opponent_score` where it returns 0 if tens digit or the ones digit is greater than 7 else returns 5;

```python
def bacon_strategy(score, opponent_score):
    """This strategy rolls 0 dice if that gives at least BACON_MARGIN points,
    and rolls BASELINE_NUM_ROLLS otherwise.

    >>> bacon_strategy(0, 0)
    5
    >>> bacon_strategy(70, 50)
    5
    >>> bacon_strategy(50, 70)
    0
    """
    "*** YOUR CODE HERE ***"
    a = opponent_score // 10
    b = opponent_score % 10
    if a >= (BACON_MARGIN - 1) or b >= (BACON_MARGIN - 1):
        return 0
    else:
        return BASELINE_NUM_ROLLS
```

> The `swap_strategy` function will take two arguments: two positive integer `score` and `opponent_score` which represent the score of two player. It returns the number of rolls base on the `opponent_score` where it returns a 0 if it would result in a beneficial swap else return `BASELINE_NUM_ROLLS`(5).

```python
def swap_strategy(score, opponent_score):
    """This strategy rolls 0 dice when it would result in a beneficial swap and
    rolls BASELINE_NUM_ROLLS if it would result in a harmful swap. It also rolls
    0 dice if that gives at least BACON_MARGIN points and rolls
    BASELINE_NUM_ROLLS otherwise.

    >>> swap_strategy(23, 60) # 23 + (1 + max(6, 0)) = 30: Beneficial swap
    0
    >>> swap_strategy(27, 18) # 27 + (1 + max(1, 8)) = 36: Harmful swap
    5
    >>> swap_strategy(50, 80) # (1 + max(8, 0)) = 9: Lots of free bacon
    0
    >>> swap_strategy(12, 12) # Baseline
    5
    """
    "*** YOUR CODE HERE ***"
    a = opponent_score // 10
    b = opponent_score % 10
    if opponent_score == 2 * (score + max(a, b) + 1):
        return 0
    elif (score + max(a, b) + 1) == 2 * opponent_score:
        return BASELINE_NUM_ROLLS
    else:
        return bacon_strategy(score, opponent_score)
```

> The `final_strategy` function will take two arguments: two positive integer `score` and `opponent_score` which represent the score of two player. It returns the number of rolls base on the `score` and `opponent_score` in different situation to maximize the winning rate of the game.

```python
def final_strategy(score, opponent_score):
    """Write a brief description of your final strategy.
    """
    "*** YOUR CODE HERE ***"
    a = opponent_score // 10
    b = opponent_score % 10
    if (opponent_score + score + max(a, b) + 1) % 7 == 0 and (max(a, b) + 1 > 4):
        return 0
    elif (opponent_score + score + max(a, b) + 1) % 7 == 0 and (max(a, b) + 1 > BACON_MARGIN):
        return bacon_strategy(score, opponent_score)
    elif score < opponent_score:
        return swap_strategy(score, opponent_score)
    elif score > opponent_score:
        return bacon_strategy(score, opponent_score)
    else:
        return BASELINE_NUM_ROLLS
```
