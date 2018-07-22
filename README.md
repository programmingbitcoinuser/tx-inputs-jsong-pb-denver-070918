
# Inputs

Each input is an output of a previous transaction. This last sentence requires more explanation as it's not intuitively obvious at first.

The inputs field can contain more than one input. This makes sense as you could use a single 100 bill to pay for a 70 dollar meal or a 50 and a 20. There are situations where there could be lots of inputs. In our analogy, we can pay for a 70 dollar meal with 14 5-dollar bills or even 7000 pennies. Each input contains 4 fields, the first two which point ot the previous transaction output and two more that define how it can be spent. These are as follows:

* Previous transaction id
* Previous transaction index
* ScriptSig
* Sequence

As explained above, each input is actually a previous transaction's output. The previous transaction id is the double_sha256 of the previous transaction's contents. This uniquely defines the previous transaction as the probability of a hash collision is very, very small. As we'll see, each transaction has to have at least 1 output, but may have many. Thus, we need to define exactly which output we're spending.

ScriptSig has to do with Bitcoin's smart contract language SCRIPT, and will be discussed more thoroughly in chapter 6. For now, think of ScriptSig as opening a lock box. Something that can only be done by the owner of the transaction output.

Sequence was originally intended as a way to do payment channels (see sidebar), but is currently used with Replace-By-Fee and Check Sequence Verify.

A couple things to note here. The amount of each input is actually not specified here. We have no idea how much is being spent unless we actually look up the transaction. Furthermore, we don't even know if we're signing the right check, so to speak, without knowing something about the previous transaction. Every node must verify that this transaction is actually signing the right check and that they're not overspending.


### Test Driven Exercise

Parse the transaction inputs


```python
from tx import Tx, TxIn
from helper import (
    little_endian_to_int,
    read_varint,
)

class Tx(Tx):

    @classmethod
    def parse(cls, s):
        '''Takes a byte stream and parses the transaction at the start
        return a Tx object
        '''
        # s.read(n) will return n bytes
        # version has 4 bytes, little-endian, interpret as int
        version = little_endian_to_int(s.read(4))

        # num_inputs is a varint, use read_varint(s)
        # each input needs parsing
        inputs = []
        # leave outputs and locktime empty for now
        outputs = []
        locktime = []
        # return an instance of the class... cls(version, inputs, outputs, locktime)
        return cls(version, inputs, outputs, locktime)


class TxIn(TxIn):

    @classmethod
    def parse(cls, s):
        '''Takes a byte stream and parses the tx_input at the start
        return a TxIn object
        '''
        # s.read(n) will return n bytes
        # prev_tx is 32 bytes, little endian
        prev_tx = s.read(32)[::-1]
        # prev_index is 4 bytes, little endian, interpret as int
        prev_index = little_endian_to_int(s.read(4))
        # script_sig is a variable field (length followed by the data)
        # get the length by using read_varint(s)
        script_sig_length = read_varint(s)
        script_sig = s.read(script_sig_length)
        # sequence is 4 bytes, little-endian, interpret as int
        sequence = little_endian_to_int(s.read(4))
        # return an instance of the class (cls(...))
        return cls(prev_tx, prev_index, script_sig, sequence)
```
