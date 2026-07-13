## Your account starts at $100. The flag costs $1,000,000. Deposits are capped. Withdrawals can't go below zero.

## No strings attached.

## We don't even need to guard the vault. This bank is impenetrable!

Provided a file as bank file

Came to know that its a executable file by using : 
- file bank
- ls -l bank
- chmod +x bank
- ./bank

Ran strings on the binary gives me the flag:
- strings bank | grep bronco

Flag : bronco{th3_b4nk_0w3s_m3_m0n3y}

