"""The Game of Hog."""

from dice import four_sided, six_sided, make_test_dice
from ucb import main, trace, log_current_line, interact

GOAL_SCORE = 100 # The goal of Hog is to score 100 points.

######################
# Phase 1: Simulator #
######################

def roll_dice(num_rolls, dice=six_sided):
    # These assert statements ensure that num_rolls is a positive integer.
    assert type(num_rolls) == int, 'num_rolls must be an integer.'
    assert num_rolls > 0, 'Must roll at least once.'
    
    # BEGIN Question 1
    
    i, total, oneflag = 0, 0, 0
    while i < num_rolls:
        one_dice_roll = dice()
        if one_dice_roll == 1:
            oneflag = True
        total += one_dice_roll
        i += 1
    if oneflag == True:
        return 1
    return total
    
    #  "END Question 1"


def take_turn(num_rolls, opponent_score, dice=six_sided):
    """Simulate a turn rolling NUM_ROLLS dice, which may be 0 (Free bacon).

    num_rolls:       The number of dice rolls that will be made.
    opponent_score:  The total score of the opponent.
    dice:            A function of no args that returns an integer outcome.
    """
    assert type(num_rolls) == int, 'num_rolls must be an integer.'
    assert num_rolls >= 0, 'Cannot roll a negative number of dice.'
    assert num_rolls <= 10, 'Cannot roll more than 10 dice.'
    assert opponent_score < 100, 'The game should be over.'
    
    # BEGIN Question 2
    
    if num_rolls == 0:
        digit_1 = opponent_score // 10   
        digit_2 = opponent_score % 10
        return max(digit_1, digit_2) + 1
    return roll_dice(num_rolls, dice)
    
    # END Question 2

def select_dice(score, opponent_score):
    """Select six-sided dice unless the sum of SCORE and OPPONENT_SCORE is a
    multiple of 7, in which case select four-sided dice (Hog wild).
    """
    
    # BEGIN Question 3
    
    if (score + opponent_score) % 7 == 0:
        return four_sided
    return six_sided
    
    # END Question 3

def is_swap(score0, score1):
    """Return True if ending a turn with SCORE0 and SCORE1 will result in a
    swap.

    Swaps occur when the last two digits of the first score are the reverse
    of the last two digits of the second score.
    """
    # BEGIN Question 4

    "Prepare scores to compare only the last two digits"
    def prepare_score(score):
        if score > 100:
            return score - 100
        return score

    "Separate score integer into digits"
    def sep_int(score):
        digit_1 = score // 10   
        digit_2 = score % 10
        return digit_1, digit_2

    score0, score1 = prepare_score(score0), prepare_score(score1)

    digit0_1, digit0_2 = sep_int(score0)
    digit1_1, digit1_2 = sep_int(score1)

    if digit0_1 == digit1_2 and digit0_2 == digit1_1:
        return True
    return False

    # END Question 4

def other(who):
    """Return the other player, for a player WHO numbered 0 or 1.

    >>> other(0)
    1
    >>> other(1)
    0
    """
    return 1 - who

def play(strategy0, strategy1, score0=0, score1=0, goal=GOAL_SCORE):
    """Simulate a game and return the final scores of both players, with
    Player 0's score first, and Player 1's score second.

    A strategy is a function that takes two total scores as arguments
    (the current player's score, and the opponent's score), and returns a
    number of dice that the current player will roll this turn.

    strategy0:  The strategy function for Player 0, who plays first
    strategy1:  The strategy function for Player 1, who plays second
    score0   :  The starting score for Player 0
    score1   :  The starting score for Player 1
    """

    who = 0  # Which player is about to take a turn, 0 (first) or 1 (second)
    # BEGIN Question 5

    while score0 < goal and score1 < goal:
        
        "Hogs Wild?"
        dice_type = select_dice(score0, score1)

        "Roll Dice then Add Roll to Total" 
        if who == 0:
            num_dice = strategy0(score0, score1)
            turn_total = take_turn(num_dice, score1, dice_type)
            score0 += turn_total
        else:
            num_dice = strategy1(score1, score0)
            turn_total = take_turn(num_dice, score0, dice_type)
            score1 += turn_total

        "Check for Swap"
        if is_swap(score0, score1) == True:
            score0, score1 = score1, score0
        
        "Change Player"
        who = other(who)

    # END Question 5
    return score0, score1

#######################
# Phase 2: Strategies #
#######################

def always_roll(n):
    """Return a strategy that always rolls N dice.

    A strategy is a function that takes two total scores as arguments
    (the current player's score, and the opponent's score), and returns a
    number of dice that the current player will roll this turn.

    >>> strategy = always_roll(5)
    >>> strategy(0, 0)
    5
    >>> strategy(99, 99)
    5
    """
    def strategy(score, opponent_score):
        return n
    return strategy

# Experiments

def make_averaged(fn, num_samples=1000):
    """Return a function that returns the average_value of FN when called.

    To implement this function, you will have to use *args syntax, a new Python
    feature introduced in this project.  See the project description.

    >>> dice = make_test_dice(3, 1, 5, 6)
    >>> averaged_dice = make_averaged(dice, 1000)
    >>> averaged_dice()
    3.75
    >>> make_averaged(roll_dice, 1000)(2, dice)
    6.0

    In this last example, two different turn scenarios are averaged.
    - In the first, the player rolls a 3 then a 1, receiving a score of 1.
    - In the other, the player rolls a 5 and 6, scoring 11.
    Thus, the average value is 6.0.
    """
    # BEGIN Question 6

    def averager(*args):
        i, total = 0, 0
        while i < num_samples:
            total += fn(*args)
            i += 1
        return total / num_samples
    return averager 

    # END Question 6

def max_scoring_num_rolls(dice=six_sided, num_samples=1000):
    """Return the number of dice (1 to 10) that gives the highest average turn
    score by calling roll_dice with the provided DICE over NUM_SAMPLES times.
    Assume that dice always return positive outcomes.

    >>> dice = make_test_dice(3)
    >>> max_scoring_num_rolls(dice)
    10
    """
    
    # BEGIN Question 7

    num_dice, best_num, best_avg = 1, 0, 0
    
    while num_dice <= 10:
        num_avg = make_averaged(roll_dice, num_samples)(num_dice, dice)
        if num_avg > best_avg:
            best_num, best_avg = num_dice, num_avg
        num_dice += 1
    return best_num

    # END Question 7

def winner(strategy0, strategy1):
    """Return 0 if strategy0 wins against strategy1, and 1 otherwise."""
    score0, score1 = play(strategy0, strategy1)
    if score0 > score1:
        return 0
    else:
        return 1

def average_win_rate(strategy, baseline=always_roll(5)):
    """Return the average win rate (0 to 1) of STRATEGY against BASELINE."""
    win_rate_as_player_0 = 1 - make_averaged(winner)(strategy, baseline)
    win_rate_as_player_1 = make_averaged(winner)(baseline, strategy)
    return (win_rate_as_player_0 + win_rate_as_player_1) / 2 # Average results

def run_experiments():
    """Run a series of strategy experiments and report results."""
    if True: # Change to False when done finding max_scoring_num_rolls
        six_sided_max = max_scoring_num_rolls(six_sided)
        print('Max scoring num rolls for six-sided dice:', six_sided_max)
        four_sided_max = max_scoring_num_rolls(four_sided)
        print('Max scoring num rolls for four-sided dice:', four_sided_max)

    if False: # Change to True to test always_roll(8)
        print('always_roll(8) win rate:', average_win_rate(always_roll(8)))

    if False: # Change to True to test bacon_strategy
        print('bacon_strategy win rate:', average_win_rate(bacon_strategy))

    if False: # Change to True to test swap_strategy
        print('swap_strategy win rate:', average_win_rate(swap_strategy))


    "*** You may add additional experiments as you wish ***"

# Strategies

def bacon_strategy(score, opponent_score, margin=8, num_rolls=5):
    """This strategy rolls 0 dice if that gives at least MARGIN points,
    and rolls NUM_ROLLS otherwise.
    """
    # BEGIN Question 8

    "*** REPLACE THIS LINE ***"
    if opponent_score > 100:
        digit_3 = 1
    else: 
        digit_3 = 0
    digit_1 = opponent_score // 10
    digit_2 = opponent_score % 10

    if max(digit_1, digit_2, digit_3) >= (margin - 1):
        return 0
    return num_rolls # Replace this statement

    # END Question 8

def swap_strategy(score, opponent_score, margin=8, num_rolls=5):
    """This strategy rolls 0 dice when it results in a beneficial swap and
    rolls NUM_ROLLS if rolling 0 dice results in a harmful swap. It also
    rolls 0 dice if that gives at least MARGIN points and rolls NUM_ROLLS
    otherwise.
    """
    
    # BEGIN Question 9

    """Compute Bacon Value"""
    if opponent_score > 100:
        digit_3 = 1
    else: 
        digit_3 = 0
    digit_1 = opponent_score // 10
    digit_2 = opponent_score % 10

    bacon = max(digit_1, digit_2, digit_3) + 1

    """1) Go for Good Swap, 2) Avoid Bad Swap, 3) Bacon Strategy"""
    if is_swap(score + bacon, opponent_score) == True and opponent_score >= (score + margin):
        return 0
    elif is_swap(score + bacon, opponent_score) == True and opponent_score < score:
        return num_rolls
    return bacon_strategy(score, opponent_score, margin, num_rolls)

    # if bacon rule is still above margin and results in neutral swap, it'll go for swap.
    # END Question 9


def final_strategy(score, opponent_score):
    """Write a brief description of your final strategy.
        1) Roll a "6" under normal circumstances, being the avg high roll.
        2) Call bacon_strategy if margin >= 9
        3) If swap is beneficial, do it.
    *** YOUR DESCRIPTION HERE ***
    """
    

    # BEGIN Question 10

    """
    Possible ideas:
    If bacon doesn't go for positive swap, roll number of dice most likely to get a swap. I should implement this but to only do that to change between options such as 3-6, whose avg roll isn't a significant drop and so it's worth losing .3 avg for 20% chance of getting a swap. I can write some code and test it.

    If roll avg of n dice is greater than best avg of 4 dice, then roll n dice most likely to give hogs wild

    Ex (for swap):
    -if score + 3 or 4 gives swap, roll 1 dice
    -if score + 6, 7, 8 gives swap, roll 2 dice
    -if score + 10, 11, roll 3
    -if score + 13, 14, 15, roll 4
    -if score + 17, 18, roll 5

    final_strategy only takes in score, opponent_score

    Solid Ideas in Order of Logic:"""

    """Compute Bacon Value"""
    
    def bacon(score):
        if score > 100:
            digit_3 = 1
        else: 
            digit_3 = 0
        digit_1 = score // 10
        digit_2 = score % 10
        bacon = max(digit_1, digit_2, digit_3) + 1
        return bacon

    """Swap Strategy Implementation"""

    flag_if_swap = is_swap(score + bacon(opponent_score), opponent_score)
    flag_good_swap = (score + bacon(opponent_score)) < opponent_score
    flag_bad_swap = (score + bacon(opponent_score)) > opponent_score

    if flag_good_swap and flag_if_swap:
        return 0

    if flag_if_swap and flag_bad_swap:
        if (score + opponent_score) % 7 == 0:
            return 4
        return 6

    """Begin: Bacon Strategy Implementation"""

    run_bacon = False

    """Part 1: Check if worth it for 4 or 6-sided die"""

    if (score + opponent_score) % 7 == 0:
        margin = 5
    else:
        margin = 9
    if bacon(opponent_score) >= margin:
        run_bacon = True

    """Part 2: Check if bacon will force Hogs Wild and if worth it"""
    margin = 5
    total_after_bacon = score + bacon(opponent_score) + opponent_score
    if total_after_bacon % 7 == 0 and bacon(opponent_score) >= margin:
        run_bacon = True

    """Part 3: Check if bacon will make opp bacon swap possible"""
    score_plus_bacon = score + bacon(opponent_score)
    new_opp_score = opponent_score + bacon(score_plus_bacon)
    if is_swap(score_plus_bacon, new_opp_score) == True:
        run_bacon = False

    """End: Choose bacon if effective"""
    if run_bacon == True:
        return 0

    """Roll Optimal Number of Dice"""
    if (score + opponent_score) % 7 == 0:
        return 4
    return 6


    """
    *Maintain Lead*
    If opponents score + 20 < your score, roll 4 or 5 dice, and lower margin by 1 or 2 for bacon strategy

    *Finish Line Strategy* 
    Something along these lines--
    If opponent is above > 78, this strategy will take precedence over swap and bacon strategy.
    (For six-dice)
    If bacon + score > 100, roll 0 dice
    If score > 96, roll 1 dice;
    If score > 92, roll 2 dice
    If score > 89, roll 3 dice
    If score > 85, roll 4 dice
    If score > 82, roll 5 dice
    If score > 78, roll 6 dice

    *Catch Up*
    If opponents score > 78 and your score  + 20 < opponents score, roll higher number dice than 6

    Roll 6--Best
    """
    
    # END Question 10


##########################
# Command Line Interface #
##########################

# Note: Functions in this section do not need to be changed.  They use features
#       of Python not yet covered in the course.


@main
def run(*args):
    """Read in the command-line argument and calls corresponding functions.

    This function uses Python syntax/techniques not yet covered in this course.
    """
    import argparse
    parser = argparse.ArgumentParser(description="Play Hog")
    parser.add_argument('--final', action='store_true',
                        help='Display the final_strategy win rate against always_roll(5)')
    parser.add_argument('--run_experiments', '-r', action='store_true',
                        help='Runs strategy experiments')
    args = parser.parse_args()

    if args.run_experiments:
        run_experiments()
    elif args.final:
        from hog_eval import final_win_rate
        win_rate = final_win_rate()
        print('Your final_strategy win rate is')
        print('    ', win_rate)
        print('(or {}%)'.format(round(win_rate * 100, 2)))
