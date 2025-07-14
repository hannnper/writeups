# L3ak CTF 2025: Whitespace
### Writeup by hannnper

The challenge included the [`flag.txt`](./flag.txt) file and the description:

> I used [this site](https://patorjk.com/software/taag/#p=display&f=3x5&t=) to make some ASCII art to save my super awesome flag, but something happened and all the spaces got deleted and I lost it. Luckily, I do remember that the MD5 hash of my flag was a7bf5f833c3e4ceff2e006ff801ec16b, so maybe you can help me.

The first 6 lines of [`flag.txt`](./flag.txt) look like this:

```

####
##
###
##
######
```

From the flag format `L3AK{...}`, we'd expect this to be an `L` but it looks like more than that. The next character would be a `3`, and the characters `L3` with whitespace removed do match these first 6 lines of [`flag.txt`](./flag.txt).

I put potential characters in a file ([`letters.txt`](./letters.txt)) and went to the site in the description and input the contents of [`letters.txt`](./letters.txt), the output of which I saved as [`ascii.txt`](./ascii.txt).

See [`code.ipynb`](./code.ipynb) for my solve code. My solution follows these steps:
- Make a dictionary mapping each character to its ascii version
- Make a dictionary of pairs of character and their ascii versions combined with spaces removed
- Split the flag file into groups of 6 lines (corresponding to letter pairs) *and make sure this matches the format of the combined character pairs from the previous step...* 🤦
- Find all possible pairs of characters that match at each position of the flag
- There was a pretty large amount of pairs, so I manually looked at which I could deduce and guessed at what the flag would say (I didn't have it exactly as some of my guesses were slightly off - e.g. I had put `T` instead of `7`). Narrowing this down to a reasonable number of possibilities at each position to try (and remembering that the flag is often 1337speak)
- Then generate all possible flags from the narrowed down character pairs
- Compare md5 hashes of possible flags to the known hash until finding one which matches

Ending up finding `L3AK{jus7_p4tt3rn_m4tch1ng_4t_f1rs7_bu7_th3n_y0u_n33d_4_h3ur1st1c_t0_n4rr0w_1t_d0wn}`!