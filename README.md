# TextCat

     /\_/\
    ( . . )
    =\_v_/=
           Perl

This is an update to the Perl version of the TextCat language guesser utility, adding UTF-8 support, new language models for longer texts and for Wikipedia search queries, language model caching for better line-by-line processing, other performance improvements, etc.

See [http://odur.let.rug.nl/~vannoord/TextCat/](http://odur.let.rug.nl/~vannoord/TextCat/) for the original Perl version.

See [https://github.com/wikimedia/wikimedia-textcat](https://github.com/wikimedia/wikimedia-textcat) for a PHP port.

## Updates

Updates from the original version include:

* updated to handle Unicode characters
* modified the output to include scores (in case we want to limit based on the score)
* pre-loaded all language models so that when processing line by line it is many times faster (a known deficiency mentioned in the comments of the original)
* put in an alphabetic sub-sort after frequency sorting of n-grams (as noted in the comments of the original, not having this is faster, but without it, results are not unique, and can vary from run to run on the same input!!), and similarly sorted outputs with the same score alphabetically.
* removed the benchmark timers (after re-shuffling some parts of the code, they weren't in a convenient location anymore, so I just took them out.

## Classification and Model Generation

See `text_cat -h` for details.

## Models

The package comes with a default language model database in the `LM` directory and a query-based language model database in the `LM-query` directory. However, model performance will depend a lot on the text corpus it will be applied to, as well as specific modifications—e.g. capitalization, diacritics, etc. Currently the library does not modify or normalize either training texts or classified texts in any way, so usage of custom language models may be more appropriate for specific applications.

Model names use [Wikipedia language codes](https://en.wikipedia.org/wiki/List_of_Wikipedias), which are often but not guaranteed to be the same as [ISO 639 language codes](https://en.wikipedia.org/wiki/ISO_639).

When detecting languages, you will generally get better results when you can limit the number of language models in use. For example, if there is virtually no chance that your text could be in Irish Gaelic, including the Irish Gaelic language model (`ga`) only increases the likelihood of mis-identification. This is particularly true for closely related languages (e.g., the Romance languages, or English/`en` and Scots/`sco`).

Limiting the number of language models used also generally improves performance. You can copy your desired language models into a new directory (and use `-d`) or specify your desired languages on the command line (use `-L`).

### Wiki-Text models

The 70 language models in `LM` are based on text extracted from randomly chosen articles from the Wikipedia for that language. The languages included were chosen based on a number of criteria, including the number of native speakers of the language, the number of queries to the various wiki projects in the language (not just Wikipedia), the list of languages supported by the original TextCat, and the size of Wikipedia in the language (i.e., the size of the collection from which to draw a training corpus).

The training corpus for each language was originally made up of ~2.7 to ~2.8M million characters, excluding markup. The texts were then lightly preprocessed. Preprocessing steps taken include: HTML Tags were removed. Lines were sorted and `uniq`-ed (so that Wikipedia idiosyncrasies—like "References", "See Also", and "This article is a stub"—are not over-represented, and so that articles randomly selected more than once were reduced to one copy). For corpora in Latin character sets, lines containing no Latin characters were removed. For corpora in non-Latin character sets, lines containing only Latin characters, numbers, and punctuation were removed. This character-set-based filtering removed from dozens to thousands of lines from the various corpora. For corpora in multiple character sets (e.g., Serbo-Croatian/`sh`, Serbian/`sr`, Turkmen/`tk`), no such character-set-based filtering was done. The final size of the training corpora ranged from ~1.8M to ~2.8M characters.

These models have not been tested and are provided as-is. New models may be added or poorly-performing models removed in the future.

These models have 4000 n-grams. The best number of n-grams to use for language identification is application-dependent. For larger texts (e.g., containing hundreds of words per sample), significantly smaller n-gram sets may be best. You can set the number to be used with `-m`.

### Wiki Query Models.

The 30 language models in `LM-query` are based on query data from Wikipedia which is less formal (e.g., fewer diacritics are used in languages that have them) and has a different distribution of words than general text. The original set of languages considered was based on the number of queries across all wiki projects for a particular week. The text has been preprocessed and many queries were removed from the training sets according to a process similar to that used on the Wiki-Text models above.

In general, query data is much messier than Wiki-Text—including junk text and queries in unexpected languages—but the overall performance on query strings, at least for English Wikipedia—is better.

The initial set of models provided was based in part on their performance on English Wikipedia queries (the first target for language ID using TextCat). For more details see our [initial report](https://www.mediawiki.org/wiki/User:TJones_%28WMF%29/Notes/Language_Detection_with_TextCat) on TextCat. More languages will be added in the future based on additional performance evaluations.

These models have 5000 n-grams. The best number of n-grams to use for language identification is application-dependent. For larger texts (e.g., containing hundreds of words per sample), significantly smaller n-gram sets may be best. For short query seen on English, French, and Spanish Wikipedia strings, a model size of 3000 n-grams has worked well. You can set the number to be used with `-m`.

