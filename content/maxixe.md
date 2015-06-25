Title: Maxixe - A simple statistical segmenter for any language
Date: 2012-01-01
Category: Code
Slug: maxixe-a-simple-statistical-segmenter-for-any
Tags: Maxixe, Ruby, Language Processing
Author: Roger Braun
Summary: Segmenting text in any language using a clever algorithm.

Recently, I wanted to use Picky for a dictionary app (and did, see WadokuJT). This worked great after fixing some small problems (which I will talk about in a later post). 

Now, to use Picky, you simply have to tell it which data to index, and then it will perform some operations on it to make it searchable. One of these operations is the tokenization: The data is split in tokens which are then indexed separately. This is rather straight-forward for western languages: Just split the string data on spaces and punctuation. So a string "I love my cat" would give you these tokens: ["I", "love", "my", "cat"].

Now, not all languages work this way. Many Asian and other languages have scriptio continua, so they just write word after word after work, with no spaces (and sometimes, no punctuation). You can even get this with western languages, for example Latin. So, if you want to tokenize text in these languages, you will have to find out where the word boundaries are.

In my case, Japanese, there are already several software packages that can split text into words, for example morphological analyzers like MeCab, ChaSen and Juman. These do more than we need, as they also analyze sentence structure and morphology of the words. Also, they are all rather large pieces of software that contain hand-written dictionaries and grammar rules to parse natural-language sentences. Further, they are often somewhat hard to install and run, the documentation often being available only in Japanese. And finally, if your language is not Japanese, but say, Chinese, or Linear B, you will have to take a completely different approach unless you want to rewrite the dictionary and grammar. I wanted something universal, written in Ruby, that I could just require from Picky.

So I went to Google Scholar and looked at the results for "japanese segmentation". I found a paper called "Mostly-unsupervised statistical segmentation of Japanese kanji sequences" by Ando and Lee. As it turns out, the algorithm described in this paper (called TANGO) is not really specific to Japanese, and you can use it to segment pretty much any language.

So i wrote a gem, Maxixe, that implements this algorithm. Here is an example of what you can do with it.

Say you have a training file containing these unsegmented sentences: 
```

    ILIKEMYDOG
    THISHOUSEISMYHOUSE
    MYDOGISSONICE
    INMYHOUSETHEREAREFOURDOGS
    IWANTAHOUSEFORMYDOG
```

You can use these to train the segmenter and split a sentence. It will detect word boundaries by itself.

```ruby
    hash = Maxixe::Trainer.generate_training_data([3,4], "dogs")

    s = Maxixe::Segmenter.new(hash)

    s.segment "MYDOGISINTHEHOUSE"
     => "MY DOG IS IN THE HOUSE"
```

Nice, isn't it? So how does this work? 

The algorithm is based on the idea that if you count any n-grams (substrings of length n) in a text, n-grams that are words or parts of word will appear more often than those which are just random strings created by two adjacent words. So what you do is use your training data to build a hash which contains the count of all n-grams in this corpus. Then, when you try to segment a string, you calculate if it is more probable that you are between two words or on a word, by comparing the count of n-grams to the left and right of the current position and the count of the n-grams that overlap with the current position. Then, after getting a numerical value for all positions, you split on all local maxima and if the value is above a certain threshold. I suggest that you read the paper yourself if you are interested. Once you grasp the underlying idea, it is quite easy to understand.

There are three things here that affect the segmentation performance: The corpus you use, the kinds of n-grams you use and your threshold. The corpus part is straightforward: Use a large amount of text that is similar to the one you want to split. The n-grams and threshold are more complicated. The above example works if you use a threshold of 0.5 and generate n-grams for n = 3,4, but you can easily find other values which will give the wrong results. Sadly, you will have to figure out which parameters are the best by segmenting a small number of sentences by hand and checking which parameters give the results most similar to your test data. I plan to add support for automating this to Maxixe, but right now, you will have to do it yourself.

And that's it! So now, if you ever need to tokenize some text that does not have spaces, give [Maxixe](https://github.com/rogerbraun/Maxixe) a try. The code is small and pure Ruby, so be sure to take a look under the hood.
