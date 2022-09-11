class: center, middle

# TidalCycles Meetup #5
## Vortex

---

# About
      
* @munshkr ([tw](https://twitter.com/munshkr), [ig](https://instagram.com/munshkr), [github](https://github.com/munshkr))

* Buenos Aires, Argentina

* Been tidalcycling since 2015 (pre-SuperDirt epoch!)

* I also <3 programming

---

# Topics

1. New features
2. New functions
3. Other changes
4. Mini-notation
5. Challenges
6. Next steps

---

# New features

* New functions imported from Tidal (and Strudel)
* Mini-notation!
* Many bugfixes
* Package published on pypi

---

# New functions

### Concatenation

- `timecat`
- `scan`

### Accumulation

- `superimpose`
- `layer`
- `iter`
- `reviter` (aka `iter'`)

### Time

- `fastgap`
- `compress`

---

# New functions

## Randomness

Implemented randomness functions, same algorithm/behavior as in Tidal.

- `rand`
- `irand`
- `perlin`
- `choose` / `choose_by`
- `wchoose` / `wchoose_by`
- `degrade` / `degrade_by`
- `undegrade` / `undegrade_by`
- `sometimes` / `sometimes_by`
- `somecycles` / `somecycles_by`

---

# New functions

## Alteration

- `range` / `rangex`

## Sampling / Structure

- `segment`
- `struct`

---

# New functions

## Arithmetic operators

Both normal and reverse order operators for all arithmetic operators were added:
`+`, `-`, `*`, `/`, `//`, `%`, `**`

```python
def __mul__(self, other):
    return self.fmap(lambda x: lambda y: x * y).app_left(reify(other))

def __rmul__(self, other):
    return self.fmap(lambda x: lambda y: y * x).app_left(reify(other))

...
```

---

# Other changes

* Make all signals *functions* instead of *variables*
  (e.g. `rand` -> `rand()`, `sine` -> `sine()`)
  - Allows to add docstrings to them
  - More consistent with other functions that return patterns (like `irand(n)`).

---

# Other changes

* Added docstrings with examples: besides documenting code and generating API
  docs automatically, you can query docs from the REPL...

```python
In [8]: rand?
Signature: rand()
Docstring:
Generate a continuous pattern of pseudo-random numbers between `0` and `1`.

>>> rand().segment(4)
File:      ~/projects/vortex/vortex/pattern.py
Type:      function

In [9]: rand??
Signature: rand()
Source:   
def rand():
    """
    Generate a continuous pattern of pseudo-random numbers between `0` and `1`.

    >>> rand().segment(4)

    """
    return signal(time_to_rand)
File:      ~/projects/vortex/vortex/pattern.py
Type:      function
```

---

# Other changes

For Pattern instance methods, evaluate `Pattern.{method}?`

```python
In [12]: Pattern.segment?
Signature: Pattern.segment(self, n)
Docstring:
Samples the pattern at a rate of `n` events per cycle.

Useful for turning a continuous pattern into a discrete one.

>>> rand().segment(4)
File:      ~/projects/vortex/vortex/pattern.py
Type:      function
```

---

# Mini-notation

* @TylerMclaughlin started working on a PEG grammar definition using
  [parsimonious](https://github.com/erikrose/parsimonious/)
* Wrote a `MiniVisitor` and `MiniInterpreter` classes that interprets the
  mininotation abstract syntax tree (AST).
* Had to define an *intermediate representation* (IR) to simplify this evaluation
  step.

---

# Mini-notation

```python
In [1]: parse_mini("bd [~ hh!!]*2 sd@2")
Out[1]: 
{'type': 'sequence',
 'elements': [{'type': 'element',
   'value': {'type': 'word', 'value': 'bd', 'index': 0},
   'modifiers': []},
  {'type': 'element',
   'value': {'type': 'polyrhythm',
    'seqs': [{'type': 'sequence',
      'elements': [{'type': 'element',
        'value': {'type': 'rest'},
        'modifiers': []},
       {'type': 'element',
        'value': {'type': 'word', 'value': 'hh', 'index': 0},
        'modifiers': [{'type': 'modifier', 'op': 'repeat', 'count': 2}]}]}]},
   'modifiers': [{'type': 'modifier', 'op': 'fast', 'value': 2}]},
  {'type': 'element',
   'value': {'type': 'word', 'value': 'sd', 'index': 0},
   'modifiers': [{'type': 'modifier', 'op': 'weight', 'value': 2}]}]}
```

---

# Mini-notation

```python
In [2]: mini("bd [~ hh!!]*2 sd@2")
Out[2]: 
~[((0, ¼), (0, ¼), 'bd'),
  (((9/32), (5/16)), ((9/32), (5/16)), 'hh'),
  (((5/16), (11/32)), ((5/16), (11/32)), 'hh'),
  (((11/32), ⅜), ((11/32), ⅜), 'hh'),
  (((13/32), (7/16)), ((13/32), (7/16)), 'hh'),
  (((7/16), (15/32)), ((7/16), (15/32)), 'hh'),
  (((15/32), ½), ((15/32), ½), 'hh'),
  ((½, 1), (½, 1), 'sd')] ...~
```

---

# Mini-notation

## Features

* Basic grammar and interpreter for simple terms (numbers, words, rests)
* Modifiers (fast, slow, degrade, replicate, weight)
* Sequences (`bd bd`)
* Polyrhythms (`[bd, sd cp*2] hh`)
* Polymeters (`{bd, sd}%2`)
* Single-cycle(?) polymeters (`<bd sd, hh*4>`)
* Euclid rhythms
* Elongate in subsequences (`@` weight op)
* Random choice in subsequences (`|` op)
* Dot pattern grouping (`bd sd . hh*4`)
* Replicate / degrade with optional parameter (`'!' int / '?' float`)
* Patternify parameters on fast, slow, and euclid

---

# Challenges

## Mini-notation

* Lack of an official grammar definition and semantics brought me doubts, but
  also opportunities to rethink Tidal's behaviour.

* It was difficult to interpret using Parsimonious `NodeVisitor`, had to create
  the IR to simplify.
  - Might be useful in other Tidal implementations?
  - Not very consistent at the moment, could be also formalized (same as grammar)

## General

* Errors in pattern functions are difficult to debug: 
  - Should we catch all errors by hand?
  - Use type hints and type enforcers, like pydantic?

---

# Ideas

## Method chaining

Strudel relies on method chaining as much as possible (inspired by Hydra?):

```javascript
freq("330 440*2").s("<sawtooth square>/2").out()
```

In Vortex you have to use the `<<` or `>>` binary operators, and pass the
pattern to a stream object with `p` (akin to `p` in Tidal, or `d1`, `d2`, etc.):

```python
p("keys", freq("0 3 5 6") >> s("<sawtooth square>/2"))
```

in Tidal this is:

```haskell
p "keys" $ freq "0 3 5 6" # s "<sawtooth square>/2"
```

---

# Ideas
## Plugins and Pattern extension

In Python you cannot dynamically add methods to a class like `Pattern`, but we
could define the Pattern dynamically at boot, with `cls` and a dynamic list of
mixin classes (multiple inheritance):

```python
mixin_classes = [SuperDirtPattern, MIDIPattern]
Pattern = type("Pattern", (BasePattern, *mixin_classes), {})
```

Python has an *entry point* mechanism for implementing plugins:

>> *Entry points are a mechanism for an installed distribution to advertise
>> components it provides to be discovered and used by other code.*

* [Entry points specification](https://packaging.python.org/en/latest/specifications/entry-points/)
* [Creating and discovering plugins](https://packaging.python.org/en/latest/guides/creating-and-discovering-plugins/)

---

# Next steps

* Documentation!
* Add more functions
* Text editor plugins? or improve internal editor

---

class: center, middle

# Thanks :-)