<img src="https://github.com/raul-dunca/assets/blob/main/.images/genetics.png?raw=true">

Given the string `CCCA CACG CAAT CAAT CCCA CACG CTGT ATAC CCTT CTCT ATAC CGTA CGTA CCTT CGCT ATAT CTCA CCTT CTCA CGGA ATAC CTAT CCTT ATCA CTAT CCTT ATCA CCTT CTCA ATCA CTCA CTCA ATAA ATAA CCTT CCCG ATAT CTAG CTGC CCTT CTAT ATAA ATAA CGTG CTTC` and knowing that the flag format start with `TFCCTF{` we can use something that is called codon table (I found out during research). Basically, mapping each of the four characters present in the given string to binary. For example `T` in binary is `01010100` and given that the string starts with `CCCA` we can see that `C = 01` and `A = 00`. Thus, I created the following script to retrieve the flag:
```python
def decode(seq):

    custom_chars = {
        'A': '00',
        'C': '01',
        'G': '10',
        'T': '11'
    }

    binary_string = ''.join([custom_chars[char] for char in seq])

    byte_chunks = [binary_string[i:i + 8] for i in range(0, len(binary_string), 8)]

    byte_array = bytearray([int(byte, 2) for byte in byte_chunks])

    return byte_array

sequence="CCCACACGCAATCAATCCCACACGCTGTATACCCTTCTCTATACCGTACGTACCTTCGCTATATCTCACCTTCTCACGGAATACCTATCCTTATCACTATCCTTATCACCTTCTCAATCACTCACTCAATAAATAACCTTCCCGATATCTAGCTGCCCTTCTATATAAATAACGTGCTTC"
decoded_bytes = decode(sequence)
print(decoded_bytes)
```

`TFCCTF{1_w1ll_g3t_th1s_4s_4_t4tt00_V3ry_s00n}`
