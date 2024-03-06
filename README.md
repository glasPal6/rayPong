# rayPong

The purpose of this is to get a basic understanding of raylib.
Most of this is based off of: https://github.com/raysan5/raylib-intro-course, definitley check that out.

## The Game

A basic, classic pong game.

The collision detection needs work. The aim is to have the ball leave with a steeper angle the further it is from the centre of the paddle.

# Building

```
mkdir build ; cd build
cp ../src/assets .
cmake ..
make
./rayPong
```
