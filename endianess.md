### What is Endianness?

Different languages read their text in different orders. English reads from left to right, for example, while Arabic is read right to left.

This is exactly what **endianness** is for computers.

If my computer **reads** bytes from left to right, and your computer reads from right to left, we're going to have issues when we need to communicate.

**Endianness** means that the bytes in computer memory are read in a certain order.

Endianness is represented two ways **Big-endian** (**BE**) and **Little-endian** (**LE**).

- **BE** stores the **big-end** first. When reading multiple bytes the first byte (or the lowest memory address) is the biggest - so it makes the most sense to people who read left to right.

Little and big endian are two ways of storing multibyte data-types ( int, float, etc).

In **little endian machines**, last byte of binary representation of the multibyte data-type is stored first. On the other hand, in big endian machines, first byte of binary representation of the multibyte data-type is stored first. 