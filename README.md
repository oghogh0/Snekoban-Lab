<h1>Snekoban Lab</h1>
<h2>Description</h2>
Sokoban, a puzzle video game designed in 1981 by Hiroyuki Imabayashi, requires the player to push boxes around in a warehouse, trying to get them to storage locations. In this lab, I implement my own version of this game. The player controls a python which can move around the board by pressing the arrow keys. Walls are stationary objects that prevent movement, and any object attempting to move to a location occupied by a wall instead remains in its original position. Computers are objects that the player can push around the board, which should be "pushed" in the same direction in which the player is moving, unless onto a wall or another computer (in which case all objects, including the player, remain in their original positions instead). Finally, targets represent stationary locations to which the player would like to push computers. The game is won when every target contains a computer. It is important to note that if there are no targets, there is no way to win the game. 
<br />

<h2>Languages and Environments Used</h2>

- <b>Python</b> 
- <b>VS code</b>

<h2>Program walk-through</h2>
<p align="left">
The DIRECTION vectors are defined as follows:<br />

direction_vector = {
    "up": (-1, 0),
    "down": (+1, 0),
    "left": (0, -1),
    "right": (0, +1),
}

<br />
<p align="left">
Create a game REPRESENTATION: <br />
This function takes in the canonical representation of the game and returns an internal representation of the game state. I chose to represent the game state as a dictionary with 2 keys: 'objects' with value which is also a dictionary of objects as keys and list of the object's positions as values, and 'features' with value which is a dictionary with row num and col num as keys, and height and width of the board as values. <br />

For example, given the game level description:<br /><br />
game_board = <br />
                [<br />
                [["wall"], ["wall"], ["wall"], ["wall"], ["wall"], ["wall"], ["wall"]],<br />
                [["wall"], ["target"], ["wall"], [], [], [], ["wall"]],<br />
                [["wall"], ["target"], ["player"], ["computer"], ["computer"], [], ["wall"]],<br />
                [["wall"], [], ["computer"], [], [], [], ["wall"]],<br />
                [["wall"], ["target"], [], [], [], [], ["wall"]],<br />
                [["wall"], ["wall"], ["wall"], ["wall"], ["wall"], ["wall"], ["wall"]],<br />
               ]<br />
    
Calling new_game(game_board) returns the dictionary:<br /><br />
{<br />
'objects': {<br />
            'wall': frozenset({(4, 0), (5, 4), (4, 6), (5, 1), (0, 2), (0, 5), (1, 0), (1, 6), <br />
                               (3, 0), (5, 0), (5, 6), (3, 6), (5, 3), (0, 1), (1, 2), (0, 4), <br />
                               (5, 2), (5, 5), (0, 0), (0, 3), (2, 0), (0, 6), (2, 6)}), <br />
            'computer': frozenset({(2, 3), (2, 4), (3, 2)}), <br />
            'target': frozenset({(1, 1), (4, 1), (2, 1)}), <br />
            'player': (2, 2)}, <br /><br />
'features': {<br />
              'row_num': 6, <br />
              'col_num': 7}<br />
}<br />
    
   
    new_rep = {}
    objects = {}
    features = {}

    objects["wall"] = []
    objects["computer"] = []
    objects["target"] = []

    features["row_num"] = len(level_description)  # height
    features["col_num"] = len(level_description[0])  # width

    for row, row_lst in enumerate(level_description):
        for col, item in enumerate(row_lst):
            if "wall" in item:
                objects["wall"].append((row, col))
            if "player" in item:
                objects["player"] = (row, col)
            if "computer" in item:
                objects["computer"].append((row, col))
            if "target" in item:
                objects["target"].append((row, col))

    objects["wall"] = frozenset(objects["wall"])  # frozenset faster than list
    objects["computer"] = frozenset(objects["computer"])
    objects["target"] = frozenset(objects["target"])

    new_rep["objects"] = objects
    # objects are keys, value is list of position, player val = tup
    new_rep["features"] = features

    return new_rep
    <br />

<p align="left">
Create a VICTORY check: <br />
This function return a Boolean value; True if the given game satisfies the victory condition, False otherwise. It is important to not that a game with no computers or targets or fewer computers than targets never satisfies the victory condition.


    computer_coord_lst = sorted(game["objects"]["computer"])
    target_coord_lst = sorted(game["objects"]["target"])

    if computer_coord_lst != []:
        return computer_coord_lst == target_coord_lst
    else:
        return False


def step_game(game, direction):
    """
    Given a game representation (of the form returned from new_game),
    return a new game representation (of that same form), representing the updated game
    after running one step of the game.  The user's input is given by
    direction, which is one of the following: {'up', 'down', 'left', 'right'}.

    This function should not mutate its input.
    """
    #copy of game
    next_game = {}

    objects = {}
    features = {}

    objects["wall"] = game["objects"]["wall"]  # dict copy
    objects["player"] = game["objects"]["player"]  # frozenset immutable

    # if on board
    if "computer" in game["objects"]:
        objects["computer"] = game["objects"]["computer"]
    if "target" in game["objects"]:
        objects["target"] = game["objects"]["target"]

    features["row_num"] = game["features"]["row_num"]
    features["col_num"] = game["features"]["col_num"]

    next_game["objects"] = objects
    next_game["features"] = features

    player_row = next_game["objects"]["player"][0]
    player_col = next_game["objects"]["player"][1]

    direct_coord = direction_vector[direction]  # tup direction assigned at start

    new_coord = (
        player_row + direct_coord[0],
        player_col + direct_coord[1],
    )  # coord after moving
    twice_coord = (player_row + 2 * direct_coord[0], player_col + 2 * direct_coord[1])

    if new_coord in next_game["objects"]["wall"]:  # direction towards wall
        return next_game  # no change
    elif (
        new_coord in next_game["objects"]["computer"]
        and twice_coord in next_game["objects"]["computer"]
    ):  # if 2 computers
        return next_game
    elif (
        new_coord in next_game["objects"]["computer"]
        and twice_coord in next_game["objects"]["wall"]
    ):  # if wall & computer
        return next_game
    else:  # can move
        if new_coord in next_game["objects"]["computer"]:  # checks if computer in front
            next_game["objects"]["computer"] = list(
                next_game["objects"]["computer"]
            )  # convert to list to mutate
            next_game["objects"]["computer"].remove(new_coord)
            next_game["objects"]["computer"].append(twice_coord)  # moves computer

        next_game["objects"]["player"] = new_coord  # moves player

        next_game["objects"]["computer"] = frozenset(
            next_game["objects"]["computer"]
        )  # convert back to frozenset

        return next_game


def dump_game(game):
    """
    Given a game representation (of the form returned from new_game), convert
    it back into a level description that would be a suitable input to new_game
    (a list of lists of lists of strings).

    This function is used by the GUI and the tests to see what your game
    implementation has done, and it can also serve as a rudimentary way to
    print out the current state of your game for testing and debugging on your
    own.
    """

    original = []

    for row in range(game["features"]["row_num"]):
        new_lst = []
        for col in range(game["features"]["col_num"]):
            new_lst.append([]) #empty position
            for key in game["objects"]:
                if (row, col) in game["objects"][key] or (row, col) == game["objects"][
                    key
                ]: 
                    new_lst[col].append(key)
        original.append(new_lst)
    return original


def get_player_computer(game):
    """
    Given a game rep, return player and computer positions
    """
    return (game["objects"]["player"], game["objects"]["computer"]) 
#tup to match visited below


def get_game(game, player, computer):
    """
    Given the player and computer position,
    returns a game
    """
    game["objects"]["player"] = player
    game["objects"]["computer"] = computer

    return game


def solve_puzzle(game):
    """
    Given a game representation (of the form returned from new game), find a
    solution.

    Return shortest list of strings representing the shortest sequence of moves ("up",
    "down", "left", and "right") needed to reach the victory condition.

    If the given level cannot be solved, return None.
    """
    #     direction_vector = {
    #     "up": (-1, 0),
    #     "down": (+1, 0),
    #     "left": (0, -1),
    #     "right": (0, +1),
    # }

    agenda = [
        ((game["objects"]["player"], game["objects"]["computer"]), [])
    ]  # tup of position, direction path
    visited = {
        (game["objects"]["player"], game["objects"]["computer"])
    }  # frozenset hashable

    while agenda:  # haven't won (means len!=0)
        current_state, current_directions = agenda.pop(0)  # agenda: (game, [""])

        if victory_check(get_game(game, current_state[0], current_state[1])):
            # next state is now current state
            # next_directions is now current_directions after appending
            return current_directions

        for direction in direction_vector:
            next_state = step_game(
                get_game(game, current_state[0], current_state[1]), direction
            )  # turn tup back to frozenset
            player_computer = get_player_computer(
                next_state
            )  # positions of (player, comp)

            if player_computer not in visited:  # player & computer position
                next_directions = current_directions + [
                    direction
                ]  # list of path of direction
                agenda.append((player_computer, next_directions))
                visited.add(player_computer)

    return None

