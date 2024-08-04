![alt text](image.png)

I used [dogbolt](https://dogbolt.org/) to decompile the given binary. The program asks for a license key and if given the correct one would yield the flag.
In the main function an exactly 17 characters string is required with the 9th one being `-`.
The rest of the license key can be considered split in two: the first 8 bytes and last 8 bytes.

For the first part, the following function must return 0:

![alt text](image-4.png)

It seems that the function performs various operations on each character after which the resulted string is compared with a variable `data_4010` and if the two match the function returns 0.
Using `BinaryNinja` we can see that `char data_4010[0x9] = "Xsl3BDxP"` so we can reverse the logic of the function:

```c++
#include <stdio.h>
#include <stdint.h>

int main() {
    char var_18[8] = {'X','s','l','3','B','D','x','P'};
    char original_var[8];

    for (int32_t i = 0; i <= 7; i += 1) {
        int32_t transformed_value = var_18[i];
        int32_t original_value = transformed_value ^ 0x33;

        int32_t rax_7 = (i % 3);

        if (rax_7 == 2)
            original_var[i] = original_value + 0x25;
        else if (rax_7 == 0)
            original_var[i] = original_value ^ 0x5a;
        else if (rax_7 == 1)
            original_var[i] = original_value - 0x10;
    }

    printf("Resulted string: ");
    for (int i = 0; i < 8; i++) {
        printf("%c", original_var[i]);
    }
    printf("\n");

    return 0;
}
```
This will return (some characters are non-printable, so I provided hex values for those): `10\x84Za\x9C\x11S` (for example: \x84 - non-printable char, but 1 byte). Looking at the main function, I realized that I could test if this is the correct first part, because of the order of ifs. If I provide a 17 bytes string with the first 8 bytes being correct and the 9th not being `-` the program must exit without a "Nope" message being printed. So I ran:
```shell
echo -e "10\x84Za\x9C\x11SX11112222" | ./license
```
Which exists as expected (notice that the 9th byte is `X`). Thus the first part was correct.

For the second part the following function must return 0:

![alt text](image-5.png)

As we can see it is somehow similar to the first function except the operations applied on the characters and the logic are different. However, again the result is compared with a variable `data_4020`, whose value is `mzXaPLzR`. After creating a cpp file and actually running the program, I find out what the function actually does, and it can be written as:

```c++
void sub_1345(char* arg1) {
    for (int i = 0; i <= 7; i++) {
        if (islower(arg1[i])) {
            arg1[i] = ((arg1[i] - 0x5c) % 0x1a) + 0x61;
        } else {
            if (isupper(arg1[i])){
            arg1[i] = ((arg1[i] - 0x30) % 0x1a) + 0x41;
            }
        }
    }
}
```
So to reverse its efect I created:

```c++
#include <iostream>
#include <cctype>
#include <cstring>

void reverse_sub_1345(char* arg1) {
    for (int i = 0; i <= 7; i++) {
        if (islower(arg1[i])) {
            arg1[i] = (((arg1[i] - 0x61)% 0x1a) +  0x5c);
        } else {
            if (isupper(arg1[i])){
            arg1[i] = ((arg1[i] - 0x41) % 0x1a) + 0x30;
            }
        }
    }
}
int main() {
    char data_4020[] = "mzXaPLzR";
    reverse_sub_1345(data_4020);

    for (int i = 0; i <= 7; i++) {
        printf("%02X ", static_cast<unsigned char>(data_4020[i]));
    }
    return 0;
}
```
This returns `huG\?;uA` which does not work because the characters `\`, `?` and `;` remain the same since the code only apply the operations on alphabetical characters (lowercase or uppercase). Since the calculation involved a % (modulo) operation, we can just add `1a` to the hex values of the special characters until we het an alphabetical character, for example:

```text
\ = 0x5c 
0x5c + 0x1a = 0x76
0x76 = v
```
Doing the same for all the special characters results in the final correct second part: `huGvYUuA`.

So what is left to do is to connect to the remote server and give the combined correct input:
```shell
echo -e "10\x84Za\x9C\x11S-huGvYUuA" | nc challs.tfcctf.com <port>
```

`TFCCTF{ac1da9096a8ad2fcb839565621bf09e892a470a6a7a0498b6259e09525096b9d}`