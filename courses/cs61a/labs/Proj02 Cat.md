- A program that measure typing speed
- With a typing autocorrect


# Overview
- 一个测试打字速度的软件
- 提供基础的打字功能, 并计算打字速度
- 提供选题功能, 根据用户选择的主题, 从文本库中生成符合该主题的打字文本
- 提供 `auto-corrector`, 开启之后为用户自动纠正错误

# Architecture
- `client-server`, server 通过 api 调用 `cats.py` 中的服务

# Phase I: typing

### 01 pick
- `selects which paragraph the user will type`
- `params`
	- `paragraphs: []string`, 所有的待选段落
	- `select: func(string)bool`, 判断某个段落是否可选
	- `k: int`, 索引, 你必须返回第 `k` 个可选的段落
- `return`
	- 选择出来符合条件的段落

### 02 about
- `params`
	- `subject: []string`, 主题列表
- `return`
	- `fn: func(paragraph: string)contains bool`, 返回一个函数, 判断该段落是否包含主题
- `idea`
	- 段落裁转成单词数组, 两次循环判断有没有相交元素
	- `remove_punctuation`
	- `split`
	- `lower`


### 03 accuracy
- `params`
	- `typed: string`, 玩家打的段落
	- `source: string`, 该段落实际上的样子
- `return`
	- 返回该玩家该段落的正确率
	- 从 `typed` 视角出发
```python
def accuracy(typed, source):
    """Return the accuracy (percentage of words typed correctly) of TYPED
    when compared to the prefix of SOURCE that was typed.

    Arguments:
        typed: a string that may contain typos
        source: a string without errors

    >>> accuracy('Cute Dog!', 'Cute Dog.')
    50.0
    >>> accuracy('A Cute Dog!', 'Cute Dog.')
    0.0
    >>> accuracy('cute Dog.', 'Cute Dog.')
    50.0
    >>> accuracy('Cute Dog. I say!', 'Cute Dog.')
    50.0
    >>> accuracy('Cute', 'Cute Dog.')
    100.0
    >>> accuracy('', 'Cute Dog.')
    0.0
    >>> accuracy('', '')
    100.0
    """
    typed_words = split(typed)
    source_words = split(source)
    # BEGIN PROBLEM 3
    "*** YOUR CODE HERE ***"
    total = len(typed_words)
    if total == 0:
        if len(source_words) == 0:
            return 100.0
        else:
            return 0.0

    count = 0
    for i in range(min(len(typed_words), len(source_words))):
        if typed_words[i] == source_words[i]:
            count += 1

    return count / total * 100
    # END PROBLEM 3

```

### 04 wpm
- `params`
	- `typed: string`, 玩家打出的段落
	- `elapsed: float`, 打字所用时间 (秒)
- `return`
	- `wpm: float`, 每分钟的打字数
- `idea`
	- `wpm` 的计算方式是将 5 个字符作为一个 `word`

# Phase II: auto correct
### 05 autocorrect
- `params`
	- `typed_word: string`, 用户刚刚输入的单词
	- `word_list: []string`, 单词列表
	- `diff_function: func(word1 string, word2 string, limit float) diff float`, 返回两个单词之间相差多少
	- `limit: float`, 最大可修正的单词差距
- `return`
	- 返回 `word_list` 中差距最小的单词
- `idea`
	- 这里有一些巧妙的做法, 利用 `max(), min() with key-function`

```python
def autocorrect(typed_word, word_list, diff_function, limit):
    """Returns the element of WORD_LIST that has the smallest difference
    from TYPED_WORD. If multiple words are tied for the smallest difference,
    return the one that appears closest to the front of WORD_LIST. If the
    difference is greater than LIMIT, instead return TYPED_WORD.

    Arguments:
        typed_word: a string representing a word that may contain typos
        word_list: a list of strings representing source words
        diff_function: a function quantifying the difference between two words
        limit: a number

    >>> ten_diff = lambda w1, w2, limit: 10 # Always returns 10
    >>> autocorrect("hwllo", ["butter", "hello", "potato"], ten_diff, 20)
    'butter'
    >>> first_diff = lambda w1, w2, limit: (1 if w1[0] != w2[0] else 0) # Checks for matching first char
    >>> autocorrect("tosting", ["testing", "asking", "fasting"], first_diff, 10)
    'testing'
    """
    # BEGIN PROBLEM 5
    "*** YOUR CODE HERE ***"
    if typed_word in word_list:
        return typed_word

    def diff_with_typed_word(word):
        return diff_function(typed_word, word, limit)

    most_close_word = min(word_list, key=diff_with_typed_word)
    if diff_with_typed_word(most_close_word) > limit:
        return typed_word
    return most_close_word
    # END PROBLEM 5

```

### 06 feline_fixes
- `params`
	- `typed: string`, 玩家输入单词
	- `source: string`, 源单词
	- `limit: number`, 最大差距
- `return`
	- `num_op: int`, 修复的操作数

### 07 minimum_mewtations
- `params`
	- `typed: string`, 玩家输入的单词
	- `source: string`, 原单词
	- `limit: int`, 修改最大差距
- `return`
	- `num_op: int`, 修复所用的操作数
- `idea`
	- 这里跟上一题的一一对应修复又有了一些不同, 需要找到最优的修复方式, 在最少操作步数下利用插入, 删除, 替换来修复
	- 直接递归爆搜
	- 注意好 `base cases`

```python
def minimum_mewtations(typed, source, limit):
    """A diff function that computes the edit distance from TYPED to SOURCE.
    This function takes in a string TYPED, a string SOURCE, and a number LIMIT.
    Arguments:
        typed: a starting word
        source: a string representing a desired goal word
        limit: a number representing an upper bound on the number of edits
    >>> big_limit = 10
    >>> minimum_mewtations("cats", "scat", big_limit)       # cats -> scats -> scat
    2
    >>> minimum_mewtations("purng", "purring", big_limit)   # purng -> purrng -> purring
    2
    >>> minimum_mewtations("ckiteus", "kittens", big_limit) # ckiteus -> kiteus -> kitteus -> kittens
    3
    """
    if limit < 0:
        return limit + 1
    if len(typed) == 0:
        return len(source)
    if len(source) == 0:
        return len(typed)
    # Recursive cases should go below here
    if typed[0] == source[0]:
        # Feel free to remove or add additional cases
        # BEGIN
        "*** YOUR CODE HERE ***"
        return minimum_mewtations(typed[1:], source[1:], limit)
        # END
    else:
        add = minimum_mewtations(typed, source[1:], limit - 1)  # Fill in these lines
        remove = minimum_mewtations(typed[1:], source, limit - 1)
        substitute = minimum_mewtations(typed[1:], source[1:], limit - 1)
        # BEGIN
        "*** YOUR CODE HERE ***"
        return 1 + min(add, remove, substitute)
        # END

```


# Phase 3: multiplayer
ok
