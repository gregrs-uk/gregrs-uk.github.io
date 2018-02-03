---
layout: post
title: "Tidytext walkthrough: correcting spellings and creating reproducible word clouds"
date: 2018-02-03
---

In this post I'll walk through the process of using [hunspell](https://github.com/ropensci/hunspell) to correct spellings automatically in a [tidytext](https://github.com/juliasilge/tidytext) analysis. We'll create a word cloud using [purrr](http://purrr.tidyverse.org)'s [`map`](http://purrr.tidyverse.org/reference/map.html) function and `set.seed` to try out various different layouts and – given the same input data – reliably reproduce our favourite.

We'll use the [GNU GPL](https://www.gnu.org/licenses/gpl-3.0.en.html) (in [plain text format](https://www.gnu.org/licenses/gpl-3.0.txt)) as our example text; it may not be the most interesting piece of prose you'll ever read (although an important one) but the process could be applied to other text such as responses to open-ended survey questions.

## Loading packages and data

First we'll load the required packages and put each individual word from the licence text into a [tibble](http://tibble.tidyverse.org) called `words`.

``` r
library(readr)
library(dplyr)
library(tidytext)

words <- tibble(line = read_lines('gpl-3.0.txt')) %>%
  # convert to lowercase later as this affects spellchecking
  unnest_tokens('word', line, to_lower = FALSE)

head(words)
```

| word    |
|:--------|
| GNU     |
| GENERAL |
| PUBLIC  |
| LICENSE |
| Version |
| 3       |

## Correcting spellings

Next we'll define a function `correct_spelling` which uses `hunspell_check` to check spellings and `hunspell_suggest` to suggest corrections, both using the British English dictionary. `hunspell_suggest` returns a list of suggestions but we just want to return the first suggestion so we pipe its output through `map(1)` and `unlist`.

Before the automatic corrections, we define any manual corrections. In this case the American spelling 'license' is recognised as valid by `hunspell_check` but I'd like to use the British spelling 'licence' so we'll add a manual case to `case_when`.

``` r
library(hunspell)
library(purrr)

correct_spelling <- function(input) {
  output <- case_when(
    # any manual corrections
    input == 'license' ~ 'licence',
    # check and (if required) correct spelling
    !hunspell_check(input, dictionary('en_GB')) ~
      hunspell_suggest(input, dictionary('en_GB')) %>%
      # get first suggestion, or NA if suggestions list is empty
      map(1, .default = NA) %>%
      unlist(),
    TRUE ~ input # if word is correct
  )
  # if input incorrectly spelled but no suggestions, return input word
  ifelse(is.na(output), input, output)
}
```

Originally I had some problems using the `correct_spelling` function with `mutate` because if there were no suggestions for an incorrectly spelled word, nothing was returned; this meant that the output vector was shorter than the input vector, causing `mutate` to fail. The `.default = NA` argument to `map` prevents this: if the suggestions list is empty it outputs `NA`, which we then catch later and change it back to the input word.

Let's see what words from the input text will be corrected by our `correct_spelling` function. We'll remove stop words first as we won't include these in our word cloud later on and it will speed up the process.

``` r
words %>%
  anti_join(stop_words, by = 'word') %>%
  rename(original = word) %>%
  group_by(original) %>%
  summarise(count = n()) %>%
  ungroup() %>% # so we can mutate word
  mutate(suggestion = correct_spelling(original)) %>%
  filter(suggestion != original)
```

| original        |  count| suggestion       |
|:----------------|------:|:-----------------|
| 6b              |      1| 6                |
| 6d              |      1| 6                |
| defenses        |      1| defences         |
| favor           |      1| favour           |
| fsf.org         |      1| forgoes          |
| Inc             |      1| Inca             |
| license         |     27| licence          |
| noncommercially |      1| non commercially |
| Sublicensing    |      1| Sub licensing    |
| WIPO            |      1| WIPE             |

Some American spellings will be converted to British ones because we're using hunspell's British English (`en_GB`) dictionary. There are also some words being corrected wrongly but as they only have one occurrence that won't be an issue for our word cloud, which will only include the most common words.

## Generating word clouds

Our `correct_spelling` function takes a while to run so we'll store the data we're going to use to generate word clouds to avoid having to correct spellings multiple times. The table below shows the 5 most common words.

``` r
cloud_words <- words %>%
  mutate(word = tolower(word)) %>%
  anti_join(stop_words, by = 'word') %>%
  # group and summarise here because correcting spellings is expensive
  group_by(word) %>%
  summarise(count = n()) %>%
  ungroup() %>% # so we can mutate word
  mutate(word = correct_spelling(word)) %>%
  # group and summarise again to combine different
  # original spellings of same word
  group_by(word) %>%
  summarise(count = sum(count))

cloud_words %>%
  arrange(desc(count)) %>%
  head(n = 5)
```

| word    |  count|
|:--------|------:|
| licence |    102|
| program |     49|
| source  |     42|
| covered |     41|
| code    |     34|

Now we'll create a word cloud to show the most common words in our text. The exact layout depends on the seed used by the random number generator, so we'll produce multiple versions by using [purrr](http://purrr.tidyverse.org)'s `map` function to set the seed before generating each version. This also means that – given the same input data – we could reproduce our favourite layout using `set.seed`.

Strictly, we don't need to pipe `cloud_words` into `wordcloud`: we could just use `cloud_words$word` instead of the `word` argument. However, I've used this method to demonstrate the use of `with`, which can help if you need to pipe into `wordcloud` following adjustments to the data earlier in the pipeline, such as `anti_join`ing some words you'd like to remove.

Below are three different possible layouts for our word cloud.

``` r
library(wordcloud)

1:3 %>% map(function(seed) {
  set.seed(seed)
  
  cloud_words %>%
    with(wordcloud(word, count, min.freq = 10,
                   colors = brewer.pal(5, 'Blues')))
})
```

![]({{ "/assets/tidytext-walkthrough-correcting-spellings-reproducible-word-clouds/wordcloud-1.png" | absolute_url }})

![]({{ "/assets/tidytext-walkthrough-correcting-spellings-reproducible-word-clouds/wordcloud-2.png" | absolute_url }})

![]({{ "/assets/tidytext-walkthrough-correcting-spellings-reproducible-word-clouds/wordcloud-3.png" | absolute_url }})

If you have any questions or suggestions, why not get in touch with me on [Twitter](https://twitter.com/gregrs_uk)?
