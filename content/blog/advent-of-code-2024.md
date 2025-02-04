+++
title = "Advent of Code 2024"
date = "2025-02-04T12:25:16+08:00"
tags = ["adventofcode",]
+++

[Advent of Code](https://adventofcode.com) is an Advent calendar of small programming puzzles released every December by [Eric Wastl](http://was.tl/). A new puzzle, consisting of two parts, is released each day from 1 Dec to 25 Dec. Participants can use any programming language to solve the puzzles.

I participated in this event for the first time in 2023. However, I was quite busy that month, so I did not complete all the puzzles. In 2024, I joined again with the goal of solving all the puzzles. From 1 Dec to 19 Dec, I solved to puzzles on the date it was released. After 19 Dec, I decided to take a break and enjoy the holidays, so I postponed the last 6 puzzles to January. Recent, I finally completed all the puzzles from Advent of Code 2024. Many thanks to Eric Wastl for creating such a fun event. Solving these puzzles has been an incredibly enjoyable experience!

After solving each puzzle, I jotted down brief notes about what I learned, how I approached the solution, or some random thoughts I had. I'd like to share some of them here.

## Finding the number of digits of a number (Day 7: Bridge Repair)

Finding the number of digits of a number is quite common is programming contest. The straight forward way to do so is to calculate the logarithm of the number. Because of my bad memory, I have to spend extra time every time to decide whether I should round up or round down the logarithm, and whether I should add 1 to the rounded logarithm.

```rust
let num: u32 = 12345;
let number_of_digit = num.checked_ilog10().unwarp() + 1; // 5
```

When I casually searched for the logarithm function in Rust standard library, I found a [Stack Overflow post](https://stackoverflow.com/questions/69297477/getting-the-length-of-an-int) mentioning an alternative way, which was kind of eye-opening to me. You can simply convert to the number into a string and get the length of the resultant string.

```rust
let num: u32 = 12345;
let number_of_digit = num.to_string().len(); // 5
```

Why I didn't find this shortcut after years of writing programs?

## Iterating over `Vec<Vec<T>>` (Day 8: Resonant Collinearity)

I used the following snippet a lot to process 2-demensional `Vec`. It converts a 2-dimensional `Vec` into an iterator that go through all entries with their indices in tuple.

```rust
let matrix = vec![
    vec!['a', 'b', 'c'],
    vec!['d', 'e', 'f'],
    vec!['g', 'h', 'i'],
];
matrix
    .iter()
    .enumerate()
    .flat_map(|(i, row)| {
        row.iter()
            .enumerate()
            .map(move |(j, value)| ((i, j), value))
    })
    .for_each(|(index, value)| println!("({}, {}): {}", index.0, index.1, value));
```

The output is:

```plaintext
(0, 0): a
(0, 1): b
(0, 2): c
(1, 0): d
(1, 1): e
(1, 2): f
(2, 0): g
(2, 1): h
(2, 2): i
```

## Entry API (Day 10: Hoof It)

I had just discovered the Entry API, which provides an idiomatic way to manipulate the contents of a collection. It helps writing more readable code.

```rust
use std::collections::HashMap;

let mut map: HashMap<&str, u32> = HashMap::new();
map.entry("my_key").and_modify(|value| { *value += 1; }).or_insert(0);
```

The last line means: Try to find the value with key "my_key". If it exists, add 1 to the found value. Otherwise, insert the default value 0 for the key "my_key".

## Transpose of a matrix in `Vec<Vec<T>>` (Day 12: Garden Groups)

```rust
let matrix = vec![vec![1, 2, 3], vec![4, 5, 6], vec![7, 8, 9]];
let height = matrix.len();
let width = matrix[0].len();

let transpose = (0..width)
    .map(|j| (0..height).map(|i| matrix[i][j]).collect::<Vec<u32>>())
    .collect::<Vec<Vec<u32>>>();
```

## Linear systems with integer variables (Day 13: Claw Contraption)

When I first saw the problem of day 13, I thought it was an integer programming problem (minimizing a linear combination subject to two linear equations on two integer variables.). However, it is a simple linear system of two integer variables.

The examples and my input do not contain any case with linear dependent equations. If it had happened, the problem would have fallen back to an integer programming problem, which is more difficult to solve. I guess these cases are not included intentionally to not make the puzzle too difficult.

The puzzle asks us find the fewest tokens required. However, since each machine is just a system of two linear-independent equations of two variables, there exists unique combination of button A and B to win the prize. So, I think the description "no more than 100 times" and "the fewest tokens" are not actual requirement. They are here is for those who try to use brute force method to figure out some implementation.

## Reminder operation v.s. Modulus operation (Day 14: Restroom Redoubt)

Be aware that `%` is the reminder operation, which is different from the modulus operation. Reminder operation and modulus operation give the same result on non-negative integers. For example, `21 % 4` gives 1 which is same as "21 mod 4". However, they act differently on negative integers. For example, `-21 % 4` means the reminder of -21 divided by 4, which is -1, while "-21 mod 4" is 3.

If you want to apply modulus operation, you can do it in the following ways.

```rust
let num: i32 = -21;
assert_eq!(((num % 4) + 4) % 4, 3); // apply % twice
assert_eq!(num.rem_euclid(4), 3);   // use rem_euclid()
```

See more on [this StackOverflow post](https://stackoverflow.com/questions/31210357/is-there-a-modulus-not-remainder-function-operation).

## Some useful packages (Day 14: Restroom Redoubt)

### Regex

```rust
let text = "x=100, y=99";
let regex = Regex::new(r"x=([0-9]+), y=([0-9]+)").unwrap();
let caps = regex.captures(text).unwrap();
assert!(caps[1].parse::<u32>(), Some(100)); // Group index starts from 1
assert!(caps[2].parse::<u32>(), Some(99));
```

### Image

```rust
let width:u32 = 100;
let height:u32 = 100;

// Create an image buffer
let mut imgbuf: ImageBuffer<Rgb<u8>, Vec<u8>> = ImageBuffer::new(width, height);
// Put a white dot in the middle
imgbuf.put_pixel(width / 2, height / 2, Rgb([255, 255, 255])); 
// Save the image into a png file
imgbuf.save("./images.png").unwrap();
```

## 3bit processor emulator (Day 17: Chronospatial Computer)

Part 1 of day 17 is to implement a 3-bit processor emulator. It is pretty straight forward, but it is really fun to do so. Part 2 is to find the initial value of a register so that the given program outputs a copy of itself. Unlike other puzzles, you have to first analyse your input (i.e. disassemble the program) before writing your solution.

The input is equivalent to the following pseudo-code:

```plaintext
while register_a > 0 {
    Generate one output, which depends on register_b, register_c
    and the last 12 bits of register_a.

    The value of register_a shifts to the right by 3 bits.
}
```

So, the first output only depends on the 1st to 12th bit of register_a, counted from the least significant bit. The second output only depends on the 4th to 15th bit of register_a, counted from the least significant bit. And so on.

My solution works in this way:

Start from a set of possible candidates, consists of all possible 12-bit numbers. Then, find out all possible 12 least significant bits of register_a that give the first correct output number, by running the program on all candidates in brute force way, to form a new set of candidates.

Next, append all possible 3-bit configurations to all candidates to update the set of candidates. Find out all possible 15 least significant bits of register_a that give the first two correct output numbers, by running the program on all candidates in brute force way, to form a new set of candidates.

Again, we append all possible 3-bit configurations to all candidates to update the set of candidates. Find out all possible 18 least significant bits of register_a that give the first three correct output numbers, by running the program on all candidates in brute force way, to form a new set of candidates.

We repeat these steps until we obtain a set of candidates that give all correct output. However, some candidates may give extra output numbers. So, we need to filter them out before giving our final answer.
