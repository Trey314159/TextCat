# TextCat

     /\_/\
    ( . . )
    =\_v_/=
           Perl

This is an update to the Perl version of the TextCat language guesser utility, adding UTF-8 support, new language models for longer texts and for Wikipedia search queries, language model caching for better line-by-line processing, other performance improvements, additional command line parameters, etc.

See [http://odur.let.rug.nl/~vannoord/TextCat/](http://odur.let.rug.nl/~vannoord/TextCat/) for the original Perl version.

See [https://github.com/wikimedia/wikimedia-textcat](https://github.com/wikimedia/wikimedia-textcat) for a PHP port.

## Updates

Updates from the original version include:

* Updated to handle Unicode characters.
* Modified the output to include scores (in case we want to limit based on the score).
* Pre-loaded all language models so that when processing line by line it is many times faster (a known deficiency mentioned in the comments of the original).
* Put in an alphabetic sub-sort after frequency sorting of n-grams (as noted in the comments of the original, not having this is faster, but without it, results are not unique, and can vary from run to run on the same input!!), and similarly sorted outputs with the same score alphabetically.
* Removed the benchmark timers (after re-shuffling some parts of the code, they weren't in a convenient location anymore, so I just took them out).
* Allow specification of "non-word" characters (`-w`), which are discarded from the input string; the default excludes numbers and spaces.
* Allow multiple language model directories (`-d`); we can use query-based models but can also fall back to Wiki-Text–based models without mixing them in one directory.
* Allow specification of a minimum input length (`-j`); shorter strings will not be identified. Mininimum length does not count non-word characters.
* Allow specification of a maximum proportion of highest (i.e., worst) possible score (`-p`), to filter "junk" texts mostly made of unknown characters and n-grams, and to a lesser extent texts in languages that are not even similar to the models in use.
* Merge n-gram count for input text and language model size to one shared value.
* Allow boosting of particular languages in results (based, for example, on *a priori* knowledge of the likelihood of various languages being present).

## Classification and Model Generation

See `text_cat -h` for details.

## Models

The package comes with a default language model database in the `LM/` directory and a query-based language model database in the `LM-query/` directory. However, model performance will depend a lot on the text corpus it will be applied to, as well as specific modifications—e.g. capitalization, diacritics, etc. Currently the library does not modify or normalize either training texts or classified texts in any way, so usage of custom language models may be more appropriate for specific applications.

Model names use [Wikipedia language codes](https://en.wikipedia.org/wiki/List_of_Wikipedias), which are often but not guaranteed to be the same as [ISO 639 language codes](https://en.wikipedia.org/wiki/ISO_639). (But see also **Wrong-Keyboard/Encoding Models** below.)

When detecting languages, you will generally get better results when you can limit the number of language models in use. For example, if there is virtually no chance that your text could be in Irish Gaelic, including the Irish Gaelic language model (`ga`) only increases the likelihood of mis-identification. This is particularly true for closely related languages (e.g., the Romance languages, or English/`en` and Scots/`sco`).

Limiting the number of language models used also generally improves performance. You can copy your desired language models into a new directory (and use `-d`) or specify your desired languages on the command line (use `-L`).

### Wiki-Text models

The 70+ language models in `LM/` are based on text extracted from randomly chosen articles from the Wikipedia for that language. The languages included were chosen based on a number of criteria, including the number of native speakers of the language, the number of queries to the various wiki projects in the language (not just Wikipedia), the list of languages supported by the original TextCat, and the size of Wikipedia in the language (i.e., the size of the collection from which to draw a training corpus).

The training corpus for each language was originally made up of ~2.7 to ~2.8M million characters, excluding markup. The texts were then lightly preprocessed. Preprocessing steps taken include: HTML Tags were removed. Lines were sorted and `uniq`-ed (so that Wikipedia idiosyncrasies—like "References", "See Also", and "This article is a stub"—are not over-represented, and so that articles randomly selected more than once were reduced to one copy). For corpora in Latin character sets, lines containing no Latin characters were removed. For corpora in non-Latin character sets, lines containing only Latin characters, numbers, and punctuation were removed. This character-set-based filtering removed from dozens to thousands of lines from the various corpora. For corpora in multiple character sets (e.g., Serbo-Croatian/`sh`, Serbian/`sr`, Turkmen/`tk`), no such character-set-based filtering was done. The final size of the training corpora ranged from ~1.8M to ~2.8M characters.

These models have not been tested and are provided as-is. New models may be added or poorly-performing models removed in the future.

These models have 10,000 n-grams. The best number of n-grams to use for language identification is application-dependent. For larger texts (e.g., containing hundreds of words per sample), significantly smaller n-gram sets may be best. You can set the number to be used with `-m`.

### Wiki Query Models.

The 30+ language models in `LM-query/` are based on query data from Wikipedia which is less formal (e.g., fewer diacritics are used in languages that have them) and has a different distribution of words than general text. The original set of languages considered was based on the number of queries across all wiki projects for a particular week. The text has been preprocessed and many queries were removed from the training sets according to a process similar to that used on the Wiki-Text models above.

In general, query data is much messier than Wiki-Text—including junk text and queries in unexpected languages—but the overall performance on query strings, at least for English Wikipedia—is better.

The initial set of models provided was based in part on their performance on English Wikipedia queries (the first target for language ID using TextCat). For more details see our [initial report](https://www.mediawiki.org/wiki/User:TJones_%28WMF%29/Notes/Language_Detection_with_TextCat) on TextCat. More languages will be added in the future based on additional performance evaluations.

These models have 10,000 n-grams. The best number of n-grams to use for language identification is application-dependent. For larger texts (e.g., containing hundreds of words per sample), significantly smaller n-gram sets may be best. For short query strings seen on English, French, and Spanish Wikipedia, model sizes of 5000 n-grams or larger have worked well. You can set the number to be used with `-m`.

### Wrong-Keyboard/Encoding Models

Five of the models provided are based on "incorrect" input types, either using the wrong keyboard, or the wrong encoding.

Wrong-keyboard input happens when someone uses two different keyboards—say Russian Cyrillic and U.S. English—and types with the wrong one active. This is reasonably common on Russian and Hebrew Wikipedias, for example. What looks like gibberish—such as *,jutvcrfz hfgcjlbz*—is actually reasonable text if the same keys are pressed on another keyboard—in this case, *богемская рапсодия* ("bohemian rapsody"). For wrong-keyboard input, the mapping between characters is one-to-one, so an existing model can be converted straightforwardly.

Wrong-encoding input happens when text is encoded using one character encoding (like [UTF-8](https://en.wikipedia.org/wiki/UTF-8)) but is interpreted as a different character encoding (such as [Windows-1251](https://en.wikipedia.org/wiki/Windows-1251)), which results in something like *Москва* ("Moscow") being rendered as *РњРѕСЃРєРІР°.* Since the character mapping is 1-to-2 (e.g., *М* → *Рњ*), the model needs to be regenerated from incorrectly encoded sample text.

The provided wrong-keyboard/encoding models are:

* `en_cyr.lm` (in both wiki-text and wiki query versions)—English as accidentally typed on a Russian Cyrillic keyboard.
* `ru_lat.lm` (in both wiki-text and wiki query versions)—Russian as accidentally typed on a U.S. English keyboard.
* `ru_win1251.lm` (only in a wiki-text version)—UTF-8 Russian accidentally interpreted as being encoded in Windows-1251.

Depending on the application, the `en_cyr` and `ru_lat` models can be used to detect non-English Latin or non-Russian Cyrillic input typed on the wrong keyboard. For example, French or Spanish typed on the Russian Cyrillic keyboard is much closer to the `en_cyr` model than it is to the Russian model.