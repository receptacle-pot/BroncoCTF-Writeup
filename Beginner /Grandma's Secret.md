## Grandma wants to protect her wifi password. Can you find out what it is before grandpa does?

<img width="2048" height="1361" alt="Letter" src="https://github.com/user-attachments/assets/60ece443-b7de-47c6-964b-9330fa5bbdaf" />


### Flag : bronco{JELLYDONUT}

This challenge combines two ciphers:
- Columnar Transposition
- ADFGVX Polybius Square

The hint tells you to undo the transposition first, then decode the ADFGVX cipher.

The keyword is: SUAGR
Sort in alphabet order : AGRSU

Since each column has 4 letters: 
- A: GVXX
- G: FVXV
- R: AFXF
- S: XVGA
- U: DAFF

Firstly in alhpabetical order : 

A G R S U

G F A X D V V F V A X X X G F X V F A F

Switch back to SUGAR order:

S U G A R

X D F G A V A V V F F F X X X A F v X F

Read off row-wise: XDFGAVAVVFGFXXXAFVXF

Split up into 2's: XD FG AV AV VF GF XX XA FV XF

Decode using given table: JELLYDONUT

So the Wi-Fi password is: JELLYDONUT

### Flag : bronco{JELLYDONUT}

