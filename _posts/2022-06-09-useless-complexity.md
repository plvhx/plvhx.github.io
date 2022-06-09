## Useless Complexity

"This problem can only be solved by writing parser module."

Said my friend at the daily meeting, and my question is:

1. There is a regular expression feature in this language, why not use that? 
2. Are you an expert parser writer?

I would rather point myself to the second question because writing a parser is such a pain in the ass and I have an experience in that, in particular.

Parser module must be fast, scalable, and must have a low-memory footprint, because every character to scan must be stored in heap, not in stack, as stack memory is function-scoped. And the heap being used must be released gracefully and systematically after doing all parsing tasks.

In my experience, writing such a parser is required a massive amount of knowledge, especially in automata theory. Need a lot of self-research on that, you can't understand enough in a couple of weeks. Pain in the ass, in conclusion.

So, don't complexify yourself for such a simple problem, please. Don't ever do that, it'll burn you.