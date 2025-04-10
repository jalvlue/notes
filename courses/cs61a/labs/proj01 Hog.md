- Project 1: The Game of Hog

# rules
- 玩家交替丢骰子
- 每次可以选择丢 `0~10` 个骰子
- 如果任意一个骰子出了 `1`, 则本轮只拿 `1` 分, 否则拿所有骰子分数总和
- 丢 `0` 个骰子举例
	- 本人个位上分数为 `m`
	- 对方十位上分数为 `n`
	- 可获得 `max(1, 3 * abs(m-n))` 分
	- 保底出一分
- 丢完骰子加分结束后, 如果分数有 `3~4` 个质因数, 则分数直接飞到下一个质数

# problems
### 00
- `dice.py`
- `make_fair_dice(sides: int)`, 生成 `1~sides` 的公平随机骰子
- `make_test_dice(seq: [])`, 生成只会在输入序列内循环的骰子

## 01
- `hog.py`
- `roll_dice(num_rolls, dice=six_sided)`
- 利用上面的各种规则, 判断这个 `dice` 抛掷 `num_rolls` 后, 获得的总分, 并返回
- 这里基本上只需要考虑 `sow_sad` 规则, 注意不能提前返回, 要准确地抛掷 `num_rolls` 次
```python
def roll_dice(num_rolls, dice=six_sided):
    """Simulate rolling the DICE exactly NUM_ROLLS > 0 times. Return the sum of
    the outcomes unless any of the outcomes is 1. In that case, return 1.

    num_rolls:  The number of dice rolls that will be made.
    dice:       A function that simulates a single dice roll outcome.
    """
    # These assert statements ensure that num_rolls is a positive integer.
    assert type(num_rolls) == int, "num_rolls must be an integer."
    assert num_rolls > 0, "Must roll at least once."
    # BEGIN PROBLEM 1
    "*** YOUR CODE HERE ***"
    sow_sad = False
    sum = 0
    i = 0
    while i < num_rolls:
        curr = dice()
        if curr == 1:
            sow_sad = True
        sum += curr
        i += 1

    return 1 if sow_sad else sum
    # END PROBLEM 1
```


### 02
- `hog.py`
- `boar_brawl(player_score, opponent_score)`
- 实现抛掷 `0` 个骰子的情况
- 根据规则简单计算
```python
def boar_brawl(player_score, opponent_score):
    """Return the points scored by rolling 0 dice according to Boar Brawl.

    player_score:     The total score of the current player.
    opponent_score:   The total score of the other player.

    """
    # BEGIN PROBLEM 2
    "*** YOUR CODE HERE ***"
    player_digit = player_score % 10
    opponent_digit = (opponent_score // 10) % 10
    return max(1, 3 * abs(player_digit - opponent_digit))
    # END PROBLEM 2
```


### 03
- `hog.py`
- `take_turn(num_rolls, player_score, opponent_score, dice=six_sided)`
- 就是计算这一轮会拿到多少分, 将前面两种规则汇聚起来
```python
def take_turn(num_rolls, player_score, opponent_score, dice=six_sided):
    """Return the points scored on a turn rolling NUM_ROLLS dice when the
    player has PLAYER_SCORE points and the opponent has OPPONENT_SCORE points.

    num_rolls:       The number of dice rolls that will be made.
    player_score:    The total score of the current player.
    opponent_score:  The total score of the other player.
    dice:            A function that simulates a single dice roll outcome.
    """
    # Leave these assert statements here; they help check for errors.
    assert type(num_rolls) == int, "num_rolls must be an integer."
    assert num_rolls >= 0, "Cannot roll a negative number of dice in take_turn."
    assert num_rolls <= 10, "Cannot roll more than 10 dice."
    # BEGIN PROBLEM 3
    "*** YOUR CODE HERE ***"
    if num_rolls == 0:
        return boar_brawl(player_score, opponent_score)

    return roll_dice(num_rolls, dice)
    # END PROBLEM 3
```


### 04
- `hog.py`
- `num_factors(n)`, 计算 `n` 有多少个质因数
- `sus_points(score)`, 对当前的 `score` 使用 `sus fuss` 规则, 直接飞到下一个质数
- `sus_update(num_rolls, player_score, opponent_score, dice=six_sided)`, 考虑所有情况, 返回一轮之后用户的总分
```python
def is_prime(n):
    """Return whether N is prime."""
    if n == 1:
        return False
    k = 2
    while k < n:
        if n % k == 0:
            return False
        k += 1
    return True


def num_factors(n):
    """Return the number of factors of N, including 1 and N itself."""
    # BEGIN PROBLEM 4
    "*** YOUR CODE HERE ***"
    if n == 1:
        return 1

    sum = 2
    i = 2
    while i < n:
        if n % i == 0:
            sum += 1
        i += 1
    return sum
    # END PROBLEM 4


def sus_points(score):
    """Return the new score of a player taking into account the Sus Fuss rule."""
    # BEGIN PROBLEM 4
    "*** YOUR CODE HERE ***"
    num = num_factors(score)
    if not num in (3, 4):
        return score

    score += 1
    while True:
        if is_prime(score):
            return score
        score += 1
    # END PROBLEM 4


def sus_update(num_rolls, player_score, opponent_score, dice=six_sided):
    """Return the total score of a player who starts their turn with
    PLAYER_SCORE and then rolls NUM_ROLLS DICE, *including* Sus Fuss.
    """
    # BEGIN PROBLEM 4
    "*** YOUR CODE HERE ***"
    return sus_points(
        player_score + take_turn(num_rolls, player_score, opponent_score, dice)
    )
    # END PROBLEM 4
```


### 05
- `def play(strategy0, strategy1, update, score0=0, score1=0, dice=six_sided, goal=GOAL)`
- 实现游戏流程
```python
def play(strategy0, strategy1, update, score0=0, score1=0, dice=six_sided, goal=GOAL):
    """Simulate a game and return the final scores of both players, with
    Player 0's score first and Player 1's score second.

    E.g., play(always_roll_5, always_roll_5, sus_update) simulates a game in
    which both players always choose to roll 5 dice on every turn and the Sus
    Fuss rule is in effect.

    A strategy function, such as always_roll_5, takes the current player's
    score and their opponent's score and returns the number of dice the current
    player chooses to roll.

    An update function, such as sus_update or simple_update, takes the number
    of dice to roll, the current player's score, the opponent's score, and the
    dice function used to simulate rolling dice. It returns the updated score
    of the current player after they take their turn.

    strategy0: The strategy for player0.
    strategy1: The strategy for player1.
    update:    The update function (used for both players).
    score0:    Starting score for Player 0
    score1:    Starting score for Player 1
    dice:      A function of zero arguments that simulates a dice roll.
    goal:      The game ends and someone wins when this score is reached.
    """
    who = 0  # Who is about to take a turn, 0 (first) or 1 (second)
    # BEGIN PROBLEM 5
    "*** YOUR CODE HERE ***"
    while score0 < goal and score1 < goal:
        if who == 0:
            num_rolls = strategy0(score0, score1)
            score0 = update(num_rolls, score0, score1, dice)
        else:
            num_rolls = strategy1(score1, score0)
            score1 = update(num_rolls, score1, score0, dice)

        who = (who + 1) % 2

    # END PROBLEM 5
    return score0, score1
```


### 06
- `def always_roll(n)`, 实现只投掷 `n` 个骰子的策略, 不管双方得分情况如何
```python
def always_roll(n):
    """Return a player strategy that always rolls N dice.

    A player strategy is a function that takes two total scores as arguments
    (the current player's score, and the opponent's score), and returns a
    number of dice that the current player will roll this turn.

    >>> strategy = always_roll(3)
    >>> strategy(0, 0)
    3
    >>> strategy(99, 99)
    3
    """
    assert n >= 0 and n <= 10
    # BEGIN PROBLEM 6
    "*** YOUR CODE HERE ***"

    def always_roll_strategy(player_score, opponent_score):
        return n

    return always_roll_strategy
    # END PROBLEM 6
```


### 07
- `def is_always_roll(strategy, goal=GOAL)`, 判断一个策略是否只产出一种 `roll`
```python
def is_always_roll(strategy, goal=GOAL):
    """Return whether STRATEGY always chooses the same number of dice to roll
    given a game that goes to GOAL points.

    >>> is_always_roll(always_roll_5)
    True
    >>> is_always_roll(always_roll(3))
    True
    >>> is_always_roll(catch_up)
    False
    """
    # BEGIN PROBLEM 7
    "*** YOUR CODE HERE ***"
    orig_roll = strategy(0, 0)
    for i in range(0, goal, 1):
        for j in range(0, goal, 1):
            curr_roll = strategy(i, j)
            if orig_roll != curr_roll:
                return False

    return True
    # END PROBLEM 7
```


### 08
- `make_averaged` 对某个函数进行采样, 然后返回期间该函数返回值的平均值
- 学到了新的 `python` 语法, 使用 `def fn(*args)` 来代表任意参数
```python
def make_averaged(original_function, samples_count=1000):
    """Return a function that returns the average value of ORIGINAL_FUNCTION
    called SAMPLES_COUNT times.

    To implement this function, you will have to use *args syntax.

    >>> dice = make_test_dice(4, 2, 5, 1)
    >>> averaged_dice = make_averaged(roll_dice, 40)
    >>> averaged_dice(1, dice)  # The avg of 10 4's, 10 2's, 10 5's, and 10 1's
    3.0
    """
    # BEGIN PROBLEM 8
    "*** YOUR CODE HERE ***"

    def average(*args):
        sum = 0
        for _ in range(samples_count):
            sum += original_function(*args)
        return sum / samples_count

    return average
    # END PROBLEM 8
```


### 09
- `def max_scoring_num_rolls(dice=six_sided, samples_count=1000)`
- 计算在 `samples_count` 尺度下, 丢几个骰子平均得分高
```python
def max_scoring_num_rolls(dice=six_sided, samples_count=1000):
    """Return the number of dice (1 to 10) that gives the highest average turn score
    by calling roll_dice with the provided DICE a total of SAMPLES_COUNT times.
    Assume that the dice always return positive outcomes.

    >>> dice = make_test_dice(1, 6)
    >>> max_scoring_num_rolls(dice)
    1
    """
    # BEGIN PROBLEM 9
    "*** YOUR CODE HERE ***"
    max_score = 0
    best_num_rolls = 0
    for i in range(1, 11):
        tmp_score = make_averaged(roll_dice, samples_count)(i, dice)
        if tmp_score > max_score:
            best_num_rolls = i
            max_score = tmp_score

    return best_num_rolls
    # END PROBLEM 9
```


### 10
- `boar_strategy`
- 实现判断投掷 `0` 是否更优的策略
```python
def boar_strategy(score, opponent_score, threshold=11, num_rolls=6):
    """This strategy returns 0 dice if Boar Brawl gives at least THRESHOLD
    points, and returns NUM_ROLLS otherwise. Ignore score and Sus Fuss.
    """
    # BEGIN PROBLEM 10
    boar_score = boar_brawl(score, opponent_score)
    if boar_score < threshold:
        return num_rolls
    return 0
    # END PROBLEM 10
```


### 11
- 在 `boar_strategy` 的基础上再考虑 `sus fuss` 规则
```python
def sus_strategy(score, opponent_score, threshold=11, num_rolls=6):
    """This strategy returns 0 dice when your score would increase by at least threshold."""
    # BEGIN PROBLEM 11
    if sus_update(0, score, opponent_score) - score < threshold:
        return num_rolls
    return 0
    # END PROBLEM 11
```


# summary
- 感觉学到了什么, 但是又感觉什么都没深入看