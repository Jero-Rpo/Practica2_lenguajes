# SI2002 Formal Languages - Assignment 2: Subset Construction

## Student information

- **Full name:** Jeronimo Restrepo Cardona
- **Class number:** C2666-SI2002-4855

## Environment

- **Operating system:** Windows 11
- **Programming language:** Python 3.12.3
- **Tools:** Python standard library only (`sys`, `re` and `collections`). No external packages are needed.

## Files

- `subset.py`: the program.
- `README.md`: this file.

## Input format

The program reads from standard input, following the format of the assignment:

1. A line with the number of cases `c`.
2. For each case:
   1. A line with the number of states `n` (the states are `1, 2, ..., n`).
   2. A line with the initial states, separated by blank spaces.
   3. A line with the alphabet, with symbols separated by blank spaces.
   4. A line with the final states, separated by blank spaces.
   5. `n` lines, one per state in order: the state followed by one set per symbol (in the order of the alphabet). Sets are written with braces, like `{1 5}`. The empty set is written as `0`.

Example (one case):

```
1
5
3 5
a b
1 4
1 {1 5} 0
2 {1} 0
3 {2 4} 0
4 0 {5}
5 {1 5} {4}
```

## How to run

Make sure Python 3 is installed. Put the input file (for example `input.txt`) in the same folder as `subset.py` and feed it to the program through standard input.

**Linux / macOS (bash or zsh):**

```bash
python3 subset.py < input.txt
```

**Windows, PowerShell** (PowerShell does not support the `<` operator, so a pipe is used):

```powershell
Get-Content input.txt | python subset.py
```

**Windows, Command Prompt (cmd.exe):**

```bat
python subset.py < input.txt
```

The program only prints to the console. To save the result in a file, redirect the output:

```bash
python3 subset.py < input.txt > output.txt
```

(In PowerShell, use `Get-Content input.txt | python subset.py | Out-File -Encoding ascii output.txt`.)

## Output format

For each case the program prints a table for the deterministic automaton M, with no extra lines (no blank lines and no messages). The first line of the table is a header with the symbols of the alphabet, followed by one row per state of M. Tables of different cases are printed one after the other.

- A state of M is named after the set of NFA states that it represents, for example `{1 2 4 5}`. The empty set is written as `0`, as in the input.
- The first column marks the initial state with `->`, the final states with `<-`, and a state that is both with `<->`.
- After the state, there is one column per symbol, in the same order as the alphabet of the input. Every cell contains one state of M.

Output for the example above:

```
              a         b
->  {3 5}     {1 2 4 5} {4}
<-  {1 2 4 5} {1 5}     {4 5}
<-  {4}       0         {5}
<-  {1 5}     {1 5}     {4}
<-  {4 5}     {1 5}     {4 5}
    0         0         0
    {5}       {1 5}     {4}
```

## Algorithm explanation

The program implements the Subset Construction presented in Kozen (1997), Lecture 6. It turns an NFA `N = (Q, Sigma, Delta, S, F)` into a DFA `M` that accepts the same language. The idea is that each state of `M` is a set of states of `N`: the states in which `N` could be at the same time after reading some input.

1. **Initial state.** The initial state of `M` is the set `S` of initial states of `N`.
2. **Transitions.** For a state `A` of `M` (a set of NFA states) and a symbol `a`, the transition is the union of `Delta(q, a)` for every `q` in `A`. This gives exactly one destination for each pair `(A, a)`, so `M` is deterministic.
3. **Exploring only reachable states.** The program keeps a queue of states that are still pending. It takes a state from the queue, computes its transition for every symbol, and when it finds a set that was not seen before, it adds it to the list of states of `M` and to the queue. It stops when the queue is empty. The sets that cannot be reached from the initial state are inaccessible states, so they are not built and do not change the language (there are at most `2^n` sets, but usually far fewer are reachable).
4. **Final states.** A state of `M` is final if it contains at least one final state of `N`.
5. **Empty set.** The empty set is a state like any other. When a union of destinations is empty, the empty set is added as a state of `M` (named `0`). Its transitions go to itself for every symbol, so the transition function of `M` is defined for every state and symbol.

Each state of `M` is processed once and, for each symbol, the program does one union of at most `n` sets. In the worst case the number of states of `M` is `2^n`.

## Implementation notes

- A state of `M` is stored as a `frozenset` of NFA states. A normal `set` cannot be used as a dictionary key because it can change, while a `frozenset` cannot.
- The transitions of the NFA are kept in a dictionary `delta[(state, symbol)]`. A missing entry is treated as the empty set.
- Blank lines in the input are ignored.
