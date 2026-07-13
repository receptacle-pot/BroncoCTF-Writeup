## Description:

I'm slowly shifting, shifting afar Char after char, char after char I'm slowly shifting (shifting afar)

And it feels like I'm fighting Underscores against the stream Braces against the stream

(Source Material: Mr. Probz, 2013)

bqmkyj{Ldfmam_Nfd_Abxjpb_Thhdqeia_Snqn_Vzey_Bok_TdudakQkwfy_Kkhxbte_Yo_Jnfvdeueqq}

### Flag : bronco{Slowly_But_Surely_Shifting_Away_Into_The_PascalSnake_Strings_Of_Characters}

This is a shift cipher where the position of each letter is subtracted by its index in the string, modulo 26.

A few edge cases:

If the resulting letter would be outside their corresponding charset (lowercases for lowercase chars, uppercases for uppercase chars), add 26 to the ASCII value so it remains in the charset. Sorta' like a within-charset modulo.

The underscores and braces remain as is, but the shift cipher continues regardless of their presence (they still count as 1 towards the index increment)

###  Flag : bronco{Slowly_But_Surely_Shifting_Away_Into_The_PascalSnake_Strings_Of_Characters}
