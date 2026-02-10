# Written Problems

## Problem 1c

Just as in the example, my own code doesn't work as intended in the second example.

My first guess for why this was the case was that, in the first attempt, Python was able to interpret the two input strings as a list of strings (or some kind of iterable, as the function's input intends). Since the code requires iterating over a string iterable, I assumed it was failing in the second exapmle because when it tried to iterate over the input (in `[self.indices[w] for w in words]`), it would fail since the input "the" is just a string and not a string iterable.

Then, I got curious about the error I was seeing, `KeyError: 'h'`. Since "h" is in the word "the", I went to check if the letters appeared in the embeddings. Apparently, "t" appears in the embeddings as a word, while "h" does not. So now, I think the error is that Python *does* consider "the" as a string iterable, but instead as a list of characters. It succeeds in indexing the embedding for "t", but fails for "h", leading to the aforementioned error.
