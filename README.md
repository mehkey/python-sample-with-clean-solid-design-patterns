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




```python


from typing import Tuple, List
from enum import Enum
from abc import ABC, abstractmethod
from unittest import TestCase

"""
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
"""

class Direction(str, Enum):
    N = 'N'
    E = 'E'
    S = 'S'
    W = 'W'

    def rotate_right(self):
        return DIRECTIONS[ (DIRECTIONS_TO_ID_DICT[self] + 1 ) % TOTAL_DIRECTIONS ]
    
    def rotate_left(self):
        return DIRECTIONS[DIRECTIONS_TO_ID_DICT[self] - 1  ]

class ChangeOption(str, Enum):
    L = 'L'
    R = 'R'
    M = 'M'

DIRECTIONS : list[Direction] = [Direction.N,Direction.E,Direction.S,Direction.W]

TOTAL_DIRECTIONS : int = len(DIRECTIONS)

"""Dictionnary of Direction to Direction Id to access the direction list"""
DIRECTIONS_TO_ID_DICT : dict[Direction,int] = { DIRECTIONS[i]:i for i in range(len(DIRECTIONS))}

POSITIONS_CHANGE : dict[Direction,list[int]]  = {
    Direction.N : [0,1] ,
    Direction.E : [1,0] ,
    Direction.S : [0,-1] ,
    Direction.W : [-1,0] 
}

X_ID : int = 0
Y_ID : int = 1

SLOW_SPEED :int = 1
FAST_SPEED :int = 2


class Position:
    """Position Class """
    def __init__(self,x:int,y:int,direction:Direction) -> None:
        self.x = x
        self.y = y
        self.direction = direction

    def get_pos_x_y(self) -> str:
        return f"{self.x},{self.y}"

    def check_bounds_is_valid(self,size: int) -> bool:
        
        if self.x < 0 \
            or self.y < 0 \
            or self.x >= size \
            or  self.y >= size:
            return False

        return True

class Change(ABC):
    """Abstract Change Command Design Pattern"""
    @abstractmethod
    def make_a_change(self,input: Position) -> None:
        pass

class Movement(Change):
    """Generic Movement"""
    def __init__(self,speed:int) -> None:
        self.speed = speed

    def make_a_change(self,input: Position) -> None:
        input.x += POSITIONS_CHANGE[input.direction][X_ID] * self.speed
        input.y += POSITIONS_CHANGE[input.direction][Y_ID] * self.speed

class MovementSlow(Movement):
    """Slow Movement"""
    def __init__(self) -> None:
        super().__init__(SLOW_SPEED)

class MovementFast(Movement):
    """Fast Movement"""
    def __init__(self) -> None:
        super().__init__(FAST_SPEED)

class MoveRight(Change):
    """Move right"""
    def make_a_change(self,input: Position) -> None:
        input.direction = input.direction.rotate_right()

class MoveLeft(Change):
    """Move left"""
    def make_a_change(self,input: Position) -> None:
        input.direction = input.direction.rotate_left()


class Player(ABC):

    """Generic Abstract Player"""
    def __init__(self, init:Tuple[int, int, Direction]):
        self.position : Position = Position(init[0],init[1],init[2])

        """Factory HashMap Calls the Template method"""
        self.change_map : dict[ChangeOption,Change] = self.setup_change_map()

    @abstractmethod
    def setup_change_map(self) -> dict[ChangeOption,Change]:
        """Template Method, returns a Dictionary of of Changet Option to Change"""
        pass

    def execute_commands(self, commands: str, arraySize : int = None) -> None:

        """ Execute a list of commands to the player and Throw Out of Bounds if needed """
        for command in commands :
            change : Change  = self._get_change_from_command(command)
            change.make_a_change(self.position)

            if arraySize and not self.position.check_bounds_is_valid(arraySize):
                raise GridOutOfBounds(self.position)

    def _get_change_from_command(self,command: str) -> Change:
        """Factory Design Pattern returning a singleton of the Change """
        if command in self.change_map:
            return self.change_map[command]
        else:
            raise CommandNotFoundException(self.change_map.keys(),command)   

    def add_position_to_set(self,positions: set[str] ) -> None:
        """Add the position to the set"""
        cur_position:str = self.position.get_pos_x_y()

        if cur_position in positions:
            raise CollisionException(cur_position)

        positions.add(cur_position)

    def remove_position_from_set(self,positions: set[str] ) -> None:
        """Remove the position from the set"""
        cur_position:str = self.position.get_pos_x_y()

        if cur_position in positions:
            positions.remove(cur_position)


"""Singleton Pattern for each Change"""
MOVE_RIGHT : Change = MoveRight()
MOVE_LEFT : Change = MoveLeft()
MOVEMENT_SLOW : Change = MovementSlow()
MOVEMENT_FAST : Change = MovementFast()

"""Factory HashMap"""
RABBIT_CHANGEOPTION_TO_CHANGE_DICT : dict[ChangeOption,Change] = {
    ChangeOption.M : MOVEMENT_FAST,
    ChangeOption.L : MOVE_LEFT,
    ChangeOption.R : MOVE_RIGHT,
}

"""Factory HashMap"""
TURTLE_CHANGEOPTION_TO_CHANGE_DICT : dict[ChangeOption,Change] = {
    ChangeOption.M : MOVEMENT_SLOW,
    ChangeOption.L : MOVE_LEFT,
    ChangeOption.R : MOVE_RIGHT
}

class Rabbit(Player):
    """Rabbit Class """
    def setup_change_map(self) -> dict[ChangeOption,Change]:
        """Template method with custom Rabbit Changes"""
        return RABBIT_CHANGEOPTION_TO_CHANGE_DICT

class Turtle(Player):
    """Turtle Class """
    def setup_change_map(self) -> dict[ChangeOption,Change]:
        """Template method with custom Turtle Changes"""
        return TURTLE_CHANGEOPTION_TO_CHANGE_DICT

class GridOutOfBounds(RuntimeError):
    """Grid Out of Bounds Runtime Error"""
    def __init__(self, position:Position):
        super().__init__(self)
        self.position = position

    def __str__(self):
        return f"The Turtle reached a bound limit at [{self.position.x},{self.position.y}] "

class CollisionException(Exception):
    """Custome Collision Exception """
    def __init__(self, position:Position):
        super().__init__(self)
        self.position = position

    def __str__(self):
        return f"The Turtles collided at position [{self.position.x},{self.position.y}] "

class CommandNotFoundException(Exception):
    """Command not found  Exception """
    def __init__(self, abailable_commands:List[ChangeOption], bad_command:ChangeOption):
        super().__init__(self)
        self.abailable_commands = abailable_commands
        self.bad_command= bad_command

    def __str__(self):
        return f"Cannot find the command. Command should be [{self.abailable_commands}] but was {self.bad_command}"

def execute_commands( 
    commands: str, 
    init:Tuple[int, int, Direction] 
) -> Tuple[int, int, Direction]:

    """Execute Commands for one Turtle"""
    turtle = Turtle(init)
    turtle.execute_commands(commands)
    return (turtle.position.x,turtle.position.y,turtle.position.direction)


def execute_commands_2(
    initial_position: Tuple[int, int, Direction],
    commands: str,
    grid_size: int,
) -> Tuple[int, int, Direction]:

    """Same as execute_commands but raises an exception if turtle ends up off the grid"""
    turtle = Turtle(initial_position)
    turtle.execute_commands(commands, grid_size)
    return (turtle.position.x,turtle.position.y,turtle.position.direction)


def execute_commands_3(
    initial_positions: List[Tuple[int, int, Direction]],
    commands: List[str],
    grid_size: int,
) -> List[Tuple[int, int, Direction]]:
    """Same as execute_commands_2 but with multiple starting turtle!"""

    turtles : Turtle = [None] * len(initial_positions)

    index :int 

    positions:set[str] = set()

    turtle:Turtle

    """Initialize the Turtles array"""
    for index, initial_position in enumerate(initial_positions):
        turtle = Turtle(initial_position)
        turtles[index] = turtle
        turtle.add_position_to_set(positions)

    """Update the positions one Command at the time and check if the turtles have the same positions in the set"""
    for move_index in range(max(map(lambda x: len(x),commands))):

        for index,turtle in enumerate(turtles):

            if move_index < len(commands[index]):

                turtle.remove_position_from_set(positions)
                turtle.execute_commands(commands[index][move_index], grid_size)
                turtle.add_position_to_set(positions)

    response: List[Tuple[int, int, Direction]] = [None] * len(initial_positions)
    
    """Create a Turtle result"""
    for index,turtle in enumerate(turtles) :
        response[index] = (turtle.position.x,turtle.position.y,turtle.position.direction)

    return response


class TurtleTests(TestCase):
    
    def test_part_1_works_execute_commands(self) -> None:
        commands = 'MMMRMMMLLL'
        initial_position = (0, 0, Direction.N)
        actual = execute_commands(commands,initial_position)
        self.assertEqual((3, 3, Direction.S), actual)
    
    def test_part_1_works_Turtle(self) -> None:
        commands = 'MMMRMMMLLL'
        initial_position = (0, 0, Direction.N)
        turtle:Turtle = Turtle(initial_position)
        turtle.execute_commands(commands)
        actual = (turtle.position.x,turtle.position.y,turtle.position.direction)
        self.assertEqual((3, 3, Direction.S), actual)
    
    def test_part_1_works_Rabbit(self) -> None:
        commands = 'MMMRMMMLLL'
        initial_position = (0, 0, Direction.N)
        rabbit:Rabbit = Rabbit(initial_position)
        rabbit.execute_commands(commands)
        actual = (rabbit.position.x,rabbit.position.y,rabbit.position.direction)
        self.assertEqual((6, 6, Direction.S), actual)
    
    def test_part_2_raises_error(self) -> None:
        test_cases = [
            ((1, 3, Direction.N), 'M', 4),
            ((3, 1, Direction.E), 'M', 4),
            ((0, 0, Direction.S), 'M', 4),
            ((0, 2, Direction.W), 'M', 4),            
        ]
        for initial_position, commands, grid_size in test_cases:
            with self.subTest(
                initial_position=initial_position,
                commands=commands,
                grid_size=grid_size
            ):
                with self.assertRaises(RuntimeError):
                    execute_commands_2(initial_position, commands, grid_size)
    
    def test_part_3_collision(self) -> None:
        initial_positions=[
            (0, 0, Direction.N),
            (0, 5, Direction.S),
        ]
        commands = ['MMM', 'MMM']
        grid_size = 5
        # Use a custom exception!
        with self.assertRaises(Exception):
            execute_commands_3(initial_positions, commands, grid_size)

    def test_part_3_works(self) -> None:
        initial_positions=[
            (0, 0, Direction.N),
            (5, 0, Direction.N),
        ]
        commands=['RMMLMMRMM', 'LMMRMMMLM']
        #There was a bug here, if this is 5, The second Turtle will start out of bounds
        grid_size = 6
        actual = execute_commands_3(initial_positions, commands, grid_size)
        expected = [(4, 2, Direction.E), (2, 3, Direction.W)]
        self.assertEqual(expected, actual)

if __name__ == '__main__':
    import unittest
    unittest.main()

```



    