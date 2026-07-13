## This text file is what happens when a chemist tries to send you a top secret message.

### Hint: all letters in the flag should be lowercase.

Provided a secret.txt file includes :
(4, 17), (2, 16), (2, 15), (4, 9), { , (3, 2, 1), (5, 3), _ , (2, 17), (3, 13, 1), (4, 5), (2, 16), (4, 17, 2), (2, 1, 2), (4, 4, 1), (2, 2, 2), _ , (3, 2, 1), (2, 2, 2), (3, 16), (3, 16), (3, 13, 1), (4, 13, 1), (2, 2, 2), (3, 16), _ , (1, 1), (3, 13, 1), (4, 5), (2, 2, 2), _ , (3, 13, 1), (4, 4, 1), _ , (2, 2, 2), (3, 17, 2), (2, 2, 2), (3, 2, 1), (2, 2, 2), (2, 15), (4, 4, 1), _ , (2, 16), (2, 17), _ , (3, 16), (9, 6), (3, 15), (4, 17, 2), (2, 1, 2), (3, 16), (2, 2, 2), }

I firestly checked if this numbers can relate to chemical elements in the periodic table.
Reading each combo of numbers as (period, group)

- (4,17) → Br, , 4th row and 17th group
- (2,16) → O 
- (2,15) → N
- (4,9) → Co

After arranging each combo of numbers in (period, group) format i got the flag

(4,17) → Br
(2,16) → O
(2,15) → N
(4,9) → Co
{
(3,2,1) → S
(5,3) → Y
_
(2,17) → F
(3,13,1) → A
(4,5) → V
(2,16) → O
(4,17,2) → r
(2,1,2) → i
(4,4,1) → T
(2,2,2) → e
_
(3,2,1) → S
(2,2,2) → e
(3,16) → S
(3,16) → S
(3,13,1) → A
(4,13,1) → G
(2,2,2) → e
(3,16) → S
_
(1,1) → H
(3,13,1) → A
(4,5) → V
(2,2,2) → e
_
(3,13,1) → A
(4,4,1) → T
_
(2,2,2) → e
(3,17,2) → l
(2,2,2) → e
(3,2,1) → S
(2,2,2) → e
(2,15) → N
(4,4,1) → T
_
(2,16) → O
(2,17) → F
_
(3,16) → S
(9,6) → Sg
(3,15) → P
(4,17,2) → r
(2,1,2) → i
(3,16) → S
(2,2,2) → e
}

Flag : bronco{my_favorite_messages_have_at_least_one_element_of_surprise}
