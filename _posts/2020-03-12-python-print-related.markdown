# Python-print related

https://ozzmaker.com/add-colour-to-text-in-python/



To make some of your text more readable, you can use [ANSI escape codes](http://en.wikipedia.org/wiki/ANSI_escape_code) to change the colour of the text output in your python program. A good use case for this is to to highlight errors.

The escape codes are entered right into the print statement.

```
print("\033[1;32;40m Bright Green  \n")
```

 

The above ANSI escape code will set the text colour to bright green. The format is;
\033[ Escape code, this is always the same
1 = Style, 1 for normal.
32 = Text colour, 32 for bright green.
40m = Background colour, 40 is for black.



This table shows some of the available formats;

| TEXT COLOR | CODE | TEXT STYLE | CODE | BACKGROUND COLOR | CODE |
| :--------- | :--- | :--------- | :--- | :--------------- | :--- |
| Black      | 30   | No effect  | 0    | Black            | 40   |
| Red        | 31   | Bold       | 1    | Red              | 41   |
| Green      | 32   | Underline  | 2    | Green            | 42   |
| Yellow     | 33   | Negative1  | 3    | Yellow           | 43   |
| Blue       | 34   | Negative2  | 5    | Blue             | 44   |
| Purple     | 35   |            |      | Purple           | 45   |
| Cyan       | 36   |            |      | Cyan             | 46   |
| White      | 37   |            |      | White            | 47   |







```
In [1]: import time

In [2]: from tqdm import tqdm

In [3]: for i in tqdm(range(10)):
   ....:     time.sleep(3)

 60%|██████    | 6/10 [00:18<00:12,  0.33 it/s]
```

Also, there is a graphical version of tqdm since [`v2.0.0`](https://github.com/tqdm/tqdm/releases/tag/v2.0.0) ([`d977a0c`](https://github.com/tqdm/tqdm/commit/d977a0c007589364c87a87198053bfe80f6cd24c)):

```
In [1]: import time

In [2]: from tqdm import tqdm_gui

In [3]: for i in tqdm_gui(range(100)):
  ....:     time.sleep(3)
```
