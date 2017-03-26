.. _pydfs-lineup-optimizer-usage:


Usage
=====

Base Usage
----------
Creating optimal lineups with **pydfs-lineup-optimizer** is very simple.
Firstly you should create object of LineupOptimizer class.
Optimizer constructor takes only one argument, it's settings.
Settings is a class inherited from BaseSettings abstract class, it store all necessary for optimizer data
like positions and budget etc. Each fantasy sport site and sport have there own settings class.
For example for FanDuel fantasy football it's FanDuelFootballSettings class.

.. code-block:: python

    from pydfs_lineup_optimizer import LineupOptimizer, FanDuelBasketballSettings


    optimizer = LineupOptimizer(FanDuelBasketballSettings)

After that you needed to load players into your optimizer. You have 2 options:
First is to load players from CSV file like this:

.. code-block:: python

    optimizer.load_players_from_CSV("path_to_csv")

Or you can load players using load_players method and pass list with players.

.. code-block:: python

    from pydfs_lineup_optimizer import Player
    optimizer.load_players(players)  # players is list of Player objects

After player loading you can create your optimal lineups with following code:

.. code-block:: python

    lineups = optimizer.optimize(n=10)

Where n is a number of lineups that you want generate.

.. note::

   Optimize method returns generator instead of list.

Example of base usage
---------------------

Below is an full example of how **pydfs-lineup-optimizer** can be used to generate optimal lineups.

.. code-block:: python

    optimizer = LineupOptimizer(settings.YahooBasketballSettings)
    optimizer.load_players_from_CSV("yahoo-NBA.csv")
    for lineup in optimizer.optimize(n=10):
        print(lineup)
        print(lineup.lineup)  # list of players
        print(lineup.fantasy_points_projection)
        print(lineup.salary_costs)

Advanced Usage
--------------

For generating optimal lineups you may need to lock some players that you want to see in your lineup.
You can do this using following code:

.. code-block:: python

    player = optimizer.get_player_by_name('Rodney Hood') # find player with specified name in your optimizer
    optimizer.add_player_to_lineup(player) # lock this player in lineup

Locked players can be unlocked as well:

.. code-block:: python

    optimizer.remove_player_from_lineup(player)

Also you can exclude some players from optimizer using this code:

.. code-block:: python

    optimizer.remove_player(player)

Excluded players can be restored using this code:

.. code-block:: python

    optimizer.restore_player(player)

You can specify maximum exposure for some players or for all players, you have several ways how to do this.
You can add "Max Exposure" column with exposure percentage for some players to csv that you will parse when load players.
Or you can set max_exposure property in Player object. If you want to set fixed exposure for all players you can
pass max_exposure parameter to optimize method

.. code-block:: python

    player = optimizer.players[0]  # get random player from optimizer players
    player.max_exposure = 0.5  # set 50% exposure

    lineups = optimizer.optimzie(n=10, max_exposure=0.3)  # set 30% exposure for all players

.. note::

    Exposure working with locked players, so if you lock some player and set exposure for 50% percentage
    this player will appears only in 50% lineups.

.. note::

   Player exposure has higher priority than max_exposure that you pass in optimize method.
   Exposure percentage rounds to ceil.

Also you can add more constraints to your optimizer like players from same team and players positions for UTIL.
For this you can pass dictionaries with constraints to optimize method.

.. code-block:: python

    team_constraint = {'OKC': 4}  # Optimizer will generate lineups with 4 Oklahoma players
    position_constraint = {'C': 1}  # Optimizer will generate lineups with >= 1 centers for util positions.
    lineups = optimizer.optimize(n=10, teams=team_constraint, positions=position_constraint)

.. note::

   Positions constraint hasn't effect for dfs sites without UTIL positions


Example of advanced usage
-------------------------

Below is an full example of how **pydfs-lineup-optimizer** can be used to generate optimal lineups with user constraints.

.. code-block:: python

    optimizer = LineupOptimizer(settings.YahooBasketballSettings)
    optimizer.load_players_from_CSV("yahoo-NBA.csv")
    nets_centers = filter(lambda p: p.team == 'Nets' and 'C' in p.positions, optimizer.players)
    for player in nets_centers:
        optimzier.remove_player(player)  # Remove all Nets centers from optimizer
    harden = optimizer.get_player_by_name('Harden')
    westbrook = optimzier.get_player_by_name('Westbrook')  # Get Harden and Westbrook
    harden.max_exposure = 0.6
    westbrook.max_exposure = 0.4  # Set exposures for Harden and Westbrook
    optimizer.add_player_to_lineup(harden)
    optimizer.add_player_to_lineup(westbrook)  # Lock Harden and Westbrook
    for lineup in optimizer.optimize(n=10, teams={'Heat': 4}, max_exposure=0.3):
        print(lineup)