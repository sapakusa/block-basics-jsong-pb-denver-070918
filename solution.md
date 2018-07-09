
# Block Basics

## Blocks

Transactions essentially transfer bitcoins from one party to another and are authorized by signatures. This definitely ensures that the sender actually authorized the transaction, but what if the sender sends the same amount to multiple people? This is called the double-spending problem and is so called because the owner of an output may try to spend the same output twice. How is the receiver to be assured that they actually received the amount?

This is where the major innovation of Bitcoin comes in with Blocks. Think of Blocks as a way to order transactions. By ordering transactions, a double-spend cannot happen as the transaction that happens later is not valid.

Now it would be really nice if we could order transactions one at a time, but that would require everyone to agree which transaction is supposed to be next and would cause a lot of overhead in coming to consensus. We can also order large batches of transactions, maybe once per day, but that wouldn't be very practical as the transactions would settle only once per day.

Bitcoin settles every 10 minutes in batches of transactions. These batches of transactions what blocks are. In this chapter we'll go through how to parse them and how to check what's called the proof of work.

### Exercise

#### What is the double_sha256 of this block? Notice anything?
```
020000208ec39428b17323fa0ddec8e887b4a7c53b8c0a0a220cfd0000000000000000005b0750fce0a889502d40508d39576821155e9c9e3f5c3157f961db38fd8b25be1e77a759e93c0118a4ffd71d
```


```python
from helper import double_sha256

hex_block = '020000208ec39428b17323fa0ddec8e887b4a7c53b8c0a0a220cfd0000000000000000005b0750fce0a889502d40508d39576821155e9c9e3f5c3157f961db38fd8b25be1e77a759e93c0118a4ffd71d'

# bytes.fromhex to get the binary
bin_block = bytes.fromhex(hex_block)
# double_sha256 the result
result = double_sha256(bin_block)
# hex() to see what it looks like
print(result.hex())
```

### Headers vs Full Blocks

Blocks are batches of transactions and the block header is metadata about the block itself. It has the following fields:

* Version
* Previous Block
* Merkle Root
* Timestamp
* Bits
* Nonce

This is the metadata for every block. The nice thing is that each field is fixed length. Version is 4 bytes, Previous Block is 32 bytes, Merkle Root is 32 bytes, Timestamp is 4 bytes, Bits is 4 bytes and Nonce is 4 bytes. The headers therefore, take up exactly 80 bytes for each block. As of this writing there are roughly 520,000 blocks so that ends up being roughl 40 megabytes. The entire blockchain, on the other hand, is roughly 150 GB, so the headers are roughly .027% of the size. This ends up becoming an important design consideration when we look at Simplified Payment Verification.

### Test Driven Exercise


```python
from io import BytesIO
from helper import (
    double_sha256,
    int_to_little_endian,
    little_endian_to_int,
)
from block import Block

class Block(Block):
    
    @classmethod
    def parse(cls, s):
        '''Takes a byte stream and parses a block. Returns a Block object'''
        # s.read(n) will read n bytes from the stream
        # version - 4 bytes, little endian, interpret as int
        version = little_endian_to_int(s.read(4))
        # prev_block - 32 bytes, little endian (use [::-1] to reverse)
        prev_block = s.read(32)[::-1]
        # merkle_root - 32 bytes, little endian (use [::-1] to reverse)
        merkle_root = s.read(32)[::-1]
        # timestamp - 4 bytes, little endian, interpret as int
        timestamp = little_endian_to_int(s.read(4))
        # bits - 4 bytes
        bits = s.read(4)
        # nonce - 4 bytes
        nonce = s.read(4)
        # initialize class
        return cls(version, prev_block, merkle_root, timestamp, bits, nonce)

    def serialize(self):
        '''Returns the 80 byte block header'''
        # version - 4 bytes, little endian
        result = int_to_little_endian(self.version, 4)
        # prev_block - 32 bytes, little endian
        result += self.prev_block[::-1]
        # merkle_root - 32 bytes, little endian
        result += self.merkle_root[::-1]
        # timestamp - 4 bytes, little endian
        result += int_to_little_endian(self.timestamp, 4)
        # bits - 4 bytes
        result += self.bits
        # nonce - 4 bytes
        result += self.nonce
        return result

    def hash(self):
        '''Returns the double-sha256 interpreted little endian of the block'''
        # serialize
        s = self.serialize()
        # double-sha256
        sha = double_sha256(s)
        # reverse
        return sha[::-1]
```
