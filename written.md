# Written Problems

## Problem 1c

Just as in the example, my own code doesn't work as intended in the second example.

My first guess for why this was the case was that, in the first attempt, Python was able to interpret the two input strings as a list of strings (or some kind of iterable, as the function's input intends). Since the code requires iterating over a string iterable, I assumed it was failing in the second exapmle because when it tried to iterate over the input (in `[self.indices[w] for w in words]`), it would fail since the input "the" is just a string and not a string iterable.

Then, I got curious about the error I was seeing, `KeyError: 'h'`. Since "h" is in the word "the", I went to check if the letters appeared in the embeddings. Apparently, "t" appears in the embeddings as a word, while "h" does not. So now, I think the error is that Python *does* consider "the" as a string iterable, but instead as a list of characters. It succeeds in indexing the embedding for "t", but fails for "h", leading to the aforementioned error.

## Problem 4a

I'm not sure whether this question requires the question accuracies to be weighted by the number of analogies in each category, but for simplicity I've calculated these answers by simply averaging all relevant categories.

### (4a) Code

I've used the following code to compute these values:

```py
embeddings = Embeddings.from_file("data/glove_XXXd.txt")
test_data = load_analogies("data/analogies.txt")

res = run_analogy_test(embeddings, test_data, k=1)

semantic_sum = 0
semantic_count = 0

syntactic_sum = 0
syntactic_count = 0

for k in res.keys():
    if k.startswith("gram"):
        # syntactic
        syntactic_sum += res[k]
        syntactic_count += 1
    else:
        # semantic
        semantic_sum += res[k]
        semantic_count += 1

print("Semantic:", semantic_sum / semantic_count)
print("Syntactic:", syntactic_sum / syntactic_count)
print("Overall:", (semantic_sum + syntactic_sum) / (semantic_count + syntactic_count))
```

### (4a) Results

| Embedding Space | Semantic | Syntactic | Overall |
|-----------------|----------|-----------|---------|
| GloVe 50        |    0.384 |     0.245 |   0.295 |
| GloVe 100       |    0.414 |     0.247 |   0.307 |
| GloVe 200       |    0.313 |     0.184 |   0.230 |

GloVe seems to consistently perform better with semantic relations than CBOW, while performing consistently worse with syntactic relations. However, GloVe seems to consistently perform worse with both semantic and syntactic relations than Skip-gram.

I would assume that as the dimensionality increases, the accuracy of the embeddings also increases. This could explain the weaker performance of our GloVe embeddings, considering that they have a lower dimensionality than both CBOW and Skip-gram; though it's interesting to note that, assuming I did the calculations correctly, GloVe 200 has performed worse than GloVe 100.

## Problem 4b

| Embedding Space | Semantic | Syntactic | Overall |
|-----------------|----------|-----------|---------|
| GloVe 50        |    0.560 |     0.504 |   0.524 |
| GloVe 100       |    0.634 |     0.628 |   0.630 |
| GloVe 200       |    0.658 |     0.634 |   0.642 |

With this added leniency, the accuracy has improved. This time, the accuracy does improve with the dimensionality, although the gains between 100 and 200 dimensions seem very small.

## Problem 4c

### (4c) Code

I've used the following code to compute these values:

```py
embeddings = Embeddings.from_file("data/glove_XXXd.txt")

print(get_closest_words(embeddings, embeddings[["italy"]] - embeddings[["france"]] + embeddings[["paris"]], k=1))
print(get_closest_words(embeddings, embeddings[["japan"]] - embeddings[["france"]] + embeddings[["paris"]], k=1))
print(get_closest_words(embeddings, embeddings[["florida"]] - embeddings[["france"]] + embeddings[["paris"]], k=1))
print(get_closest_words(embeddings, embeddings[["small"]] - embeddings[["big"]] + embeddings[["bigger"]], k=1))
print(get_closest_words(embeddings, embeddings[["cold"]] - embeddings[["big"]] + embeddings[["bigger"]], k=1))
print(get_closest_words(embeddings, embeddings[["quick"]] - embeddings[["big"]] + embeddings[["bigger"]], k=1))
```

### (4c) Results

| Analogy Question                | Gold Answer  | GloVe 50 | GloVe 100 | GloVe 200 |
|---------------------------------|--------------|----------|-----------|-----------|
| france : paris :: italy : $x$   | rome         | rome     | rome      | rome      |
| france : paris :: japan : $x$   | tokyo        | tokyo    | tokyo     | tokyo     |
| france : paris :: florida : $x$ | tallahassee  | miami    | florida   | florida   |
| big : bigger :: small : $x$     | smaller      | larger   | larger    | smaller   |
| big : bigger :: cold : $x$      | colder       | cold     | cold      | cold      |
| big : bigger :: quick : $x$     | quicker      | quick    | quick     | quick     |

The embedding spaces seem to have more success here with semantic analogies over syntactic ones. All models did well with "rome" and "tokyo", though they struggled with "tallahassee". It was interesting how GloVe 50 gave "miami" for "florida", which at least makes some sense since it's the biggest city in Florida (or at least part of the largest metropolitan area, according to Google), while the larger embeddings just gave "florida". All embedding sizes struggled with the syntactic analogies, with the only correct one being by GloVe 200 for "small".

The results in the paper are better overall. The paper reports that "tallahassee" was correctly deduced, and the results are vastly improved for the syntactic analogies, correctly reporting "colder" and "quicker". It's interesting that those results also yielded "larger" instead of "smaller", which is the same mistake that GloVe 50 and 100 have made.
