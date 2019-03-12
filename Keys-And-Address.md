# How Cryptonote Addresses Are Created


Cryptonote Public Addresses differ in several ways compared to Bitcoin. First, Cryptonote uses two keypairs, known as the spend keypair and the view keypair. Furthermore, these keys are EdDSA (specifically ed25519) keys, whereas Bitcoin uses ECDSA (specifically secp256k1) keys. Finally, Cryptonote Public Addresses are direct representations of the pair of public keys, whereas Bitcoin (and clones) uses a hash of the single public key. EdDSA keys (both private and public) are 256 bits long, or 64 hexadecimal characters. Not every 256-bit integer is a valid EdDSA scalar (private key); it must be less than the "curve order". The function to do this is labeled sc_reduce32. 


To add to the confusion, there are presently at least three different methods of private key derivation in existence for Monero (and other Cryptonotes), though Bitcoin also has many:

1. ***Original (non-deterministic) Style*** – The Private Spend Key and Private View Key are both independently and randomly chosen to form an account. You can simulate this above by pressing the "Random" buttons next to fields 3. and 4., then pressing "Gen 5.", "Gen 6.", and "Gen 7.", in that order. There is no good way to back up a non-deterministic account other than keeping copies of the files; you need to have a copy of both private keys, but presently only MyMonero will accept the two keys as input instead of a seed/wallet file. For these reasons, it is not recommended to use an account of this type.
2. ***Mnemonic (Electrum or Deterministic) Style*** – In this style, the Private View Key is derived from the Private Spend Key, so you only need to remember one thing: the seed, which is actually just a representation of the Private Spend Key itself. This 256-bit scalar can be easily converted to a "24-digit" Base1626 "number" in the form of a mnemonic seed, which is 25 words long with the last word being used as a checksum. Mnemonics convert on a ratio of 4:3 minimum: four bytes creates three words, plus one checksum word; eight bytes creates six words, plus one checksum word; and so on. The "seeds" created by this method will always be valid scalars as they are sent to sc_reduce32 first. The Private View Key is derived by hashing the Private Spend Key with Keccak-256, producing a second 256-bit integer, which is then sent to sc_reduce32. You can test out this style above by pressing the "Random" button on the upper right, or by pressing either of the "Random" buttons next to fields 1. and 2., then the various "Gen x." buttons. You can backup accounts of this type by writing down or otherwise saving the 25 word deterministic seed; you can easily restore using both Simplewallet and MyMonero.
3. ***MyMonero Style*** – This is similar to 2., but uses a 13 word seed instead of a 25 word seed. The 13 words convert to a 128-bit integer that is used for both spend and view key derivation, in the following form: the 128-bit integer is hashed with Keccak-256 to produce a 256-bit integer, a. a is sent to sc_reduce32, which returns the Private Spend Key. a is hashed once more with Keccak-256 to produce a second 256-bit integer, b. b is then sent to sc_reduce32, which returns the Private View Key. You may have noticed a critical difference between this style and the Electrum Style: MyMonero's Private View Key derivation is done by hashing random integer a, while Electrum Style derivation is done by hashing the Private Spend Key. This means that 13 and 25 word seeds are not compatible – it is not possible to create an Electrum Style seed (and account) that matches a MyMonero Style seed (and account) or vice versa; the view keypair will always be different. You can test out this style above with the "Random MyMonero" button. To backup MyMonero accounts, save the 13 word seed; you can currently "restore" using MyMonero only (you're really just logging in) – Simplewallet does not currently support 13-word seeds.


Above we discussed the different ways private keys are derived; the rest of the address generation process is the same in all three cases. The Private Spend Key and Private View Key are sent to the ed25519 scalarmult function to create their counterparts, the Public Spend Key and Public View Key. To create the actual Public Address, the following is performed:

1. The pair of public keys are prepended with one network byte (the number 18, 0x12, for Monero). It looks like this: (network byte) + (32-byte public spend key) + (32-byte public view key).
2. These 65 bytes are hashed with Keccak-256.
3. The first four bytes of the hash from 2. are appended to 1., creating a 69-byte Public Address.
4. As a last step, this 69-byte string is converted to Base58. However, it's not done all at once like a Bitcoin address, but rather in 8-byte blocks. This gives us eight full-sized blocks and one 5-byte block. Eight bytes converts to 11 or less Base58 characters; if a particular block converts to <11 characters, the conversion pads it with "1"s (1 is 0 in Base58). Likewise, the final 5-byte block can convert to 7 or less Base58 digits; the conversion will ensure the result is 7 digits. Due to the conditional padding, the 69-byte string will always convert to 95 Base58 characters (8 * 11 + 7).
5. This 95-character result is the (obscenely long) Cryptonote Public Address!
6. If you're creating an integrated address, simply append the 64-bit payment ID to step 1 and continue; everything else is the same except for the lengths (77 bytes total, 106 Base58 digits) and the prepended byte (19, 0x13).

# online generator

[generator](https://xmr.llcoins.net/addresstests.html)



# Cryptonote Key And Address
[https://cryptonote.org/cns/cns007.txt](https://cryptonote.org/cns/cns007.txt)


翻译：

# CryptoNote密钥与地址

## 摘要

这个文档是CryptoNote标准（一个点对点的匿名支付系统）的一部分。它定义了不同的在CryptoNote里使用的用户密钥，以及如何将地址编码成为一个字符数字组成的字符串。

## 1. 介绍
CryptoNote标准在两个场景里会提到`密钥`:
1. 在交易中使用的一次性私钥或者公钥。txout_to_key是CryptoNote基本的交易类型。
2. 用户永久保存在个人的CryptoNote钱包的密钥。它们用来检测进来的交易和承继环签名里的一次性私钥。

每个用户默认有两个永久密钥对（即同时包含公钥，私钥）。这两个密钥对的公钥部分组成了用户的地址。

## 2. 定义

公共密钥：数字签名校验时用于区分一个独立用户的数据。
私有密钥：只被一个独立用户所拥有，从而让他可以创建属于他的数字签名。
   



