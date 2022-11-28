# Turtle on a board
You are given the following:

an m x m square grid
a Turtle that:
has a facing direction
is placed on the bottom left of the grid facing North, assume these this corresponds to coordinates x, y, d = (0, 0, 'N')
There are 4 directions:

'N': for North (i.e. up)
'E': for East (i.e. right)
'S': for South (i.e. down)
'W': for West (i.e. left)

## Part 1
Suppose the grid is infinite (i.e. M = infinity) and the turtle's starting position is (0, 0, 'N') Given an input string that looks like 'MLR' where:

'M' = move forward 1 cell in the direction the turtle is facing

'L' = turn left (no move, i.e. stays on the same cell)

'R' = turn right (no move, i.e. stays on the same cell)

input: 'MMMRMMMLLL' output: (3, 3, 'S')

## Part 2
Suppose now that the grid is finite and that you are given it's dimension m. We would like to throw an error if the turtle "falls off" the grid. Modifiy your previous impelemtation to support this functionality

## Part 3
Suppose now that we start with multiple turtles on the same finite grid. Each turtle will have a starting position and direction and they will move sequentially in turn.






V3 of the Turtle and Rabbit Grid Problem following the description on README.md

    This version includes:
        - Direction Enum
        - ChangeOption Enum 
        - Change Class  using the Command Design Pattern and subclasses
        - Position Class
        - Player Class using the Template Design Pattern and Rabbit, Turtle Subclass
        - A factory Design pattern
        - execute_command_1() for 1 Turtle
        - execute_command_2() to check if a turtle is out of bounds 
        - execute_command_3() for a Group of Turtle to Check Collisions
        - GridOutOfBounds custom RuntimeError 
        - CollisionException custom Exception

    This version includes the following Design patterns
        - Singleton Design Pattern
        - Factory Design Pattern
        - Command Design Pattern
        - Template Design Pattern
        - Open Close Principle where a change is Open for Extension and closed for modification
        - Single Responsibility Principle

    Author: @Meki