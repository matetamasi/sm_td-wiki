This is a user library for QMK with custom implementations of Tap Dance functions. 
It offers smooth, fast and reliable tap dance functions for your keyboard.
Base functions are:
- better tap+tap vs hold+tap interpretation with two different keys
- better multi-tap and hold (and tap again then) interpretation of the same key
- more reactive response on multiple taps (and holds)

This library is created because standard Tap Dance library in QMK and MT/ LT functions are not responsive enough.
This standard libraries you sometimes have to wait to make sure that QMK handles your press as hold, sometimes there are unexpected interpretations, sometimes you are forced to pay too much attention on how to release keys. That irritated me much, so I've built custom users library to overcome that problems.

Core idea of that library is making decision about holding or tapping based on difference between key release. As human we don't pay much attention on a way we release a key. We are used to press keys in certain sequence but not releasing in a sequence, so we tend to hold a key for longer time then it's necessary to. For a human Seq. A and Seq. B from example below are almost the same and should be processed the same way, but standard library will interpret Seq. A as hold+tap, and Seq. B and C. as tap+tap. In contrast sm_td will interpret A. and B. as hold+tap, and C. as tap+tap.

```
         |     Sequence A.     |     Sequence B.     |      Sequence C.    |
0ms  - - | - ┌—————┐ - - - - - | - ┌—————┐ - - - - - | - ┌—————┐ - - - - - |
         |   │macro│           |   │macro│           |   │macro│           |
         |   │ key │           |   │ key │           |   │ key │           |
         |   │     │           |   │     │           |   │     │           |
10ms - - | - │     │ ┌—————┐ - | - │     │ ┌—————┐ - | - │     │ ┌—————┐ - |
         |   │     │ │foll.│   |   │     │ │foll.│   |   │     │ │foll.│   |
         |   │     │ │ key │   |   │     │ │ key │   |   │     │ │ key │   |
         |   │     │ │     │   |   │     │ │     │   |   │     │ │     │   |
         |   │     │ │     │   |   │     │ │     │   |   │     │ │     │   |
         |   │     │ │     │   |   │     │ │     │   |   │     │ │     │   |
         |   │     │ │     │   |   │     │ │     │   |   │     │ │     │   |
49ms - - | - │     │ └—————┘ - |   │     │ │     │   |   │     │ │     │   |
50ms - - | - └—————┘ - - - - - | - └—————┘ │     │ - | - └—————┘ │     │ - |
51ms - - | - - - - - - - - - - | - - - - - └—————┘ - |           │     │   |
         |                     |                     |           │     │   |
         |                     |                     |           │     │   |
         |                     |                     |           │     │   |
         |                     |                     |           │     │   |
200ms  - | - - - - - - - - - - | - - - - - - - - - - | - - - - - └—————┘ - |  
```


