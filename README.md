# Scammer LGS Stats Script
This is the poorly written script used to aggregate all match result data after every match. 

## Setup
First install Python https://www.python.org/downloads/

Next simply download the zip and extract the files. The only necessary files in the folder that you want to work from are
 
glicko2.py 
- Library used for calculating player glicko scores
- Credit: https://github.com/sublee/glicko2
- Further reading: http://www.glicko.net/glicko/glicko.pdf

script.py

results.csv
 - File that contains player data

## Usage
Open up command line and navigate to the folder where the files are present
```
cd folder path
```
Populate player data file
 - Example showing the correct format 
 - PlayerA,PlayerB,Result (1 if PlayerA won, 0 if PlayerB won),Set,Date
 - A line with all 5 elements tells the script to begin tracking a new event
 - Otherwise the script will assume that the match line it is reading is from the most recently specified event
 - Make sure to have empty lines between events
```
Simon,Chris A,0,RNA,9-30-23
Juwan,Jacob,0
Grant,Matt,0
Simon,Jacob,0
Chris A,Grant,1

Jacob,Luca,0,NEO,9-30-23
Nick R,Juwan,0
Karen,Chris A,0
Chris A,Juwan,0
Karen,Nick R,0
Luca,Matt,0
Simon,Jacob,1
Nick R,Simon,1
```
Run the command for executing the script
```
py script.py
```

Upon executing, there should be a new file generated called "data.csv".

## Explanation of data file
### ELO
Popular rating system for tracking skill level of a player. ELO has limitations which were addressed by Glicko years later. 
Can be configured by altering the "K value" variable which effects how volatile score changes are after matches (how much a ELO a player will win or lose after matches).
Change the 3rd variable (default set to 30) to experiment with how drastic ELO changes should be. (Line 155)
```
gameResult = EloRating(elo_dict[player1][0], elo_dict[player2][0], 30, result.strip())
```  
### Glicko
Variations of the Glicko rating system are used for recording player strength in many online competitive games. 
Rather than assigning a simple number rating to players, Glicko adds a new field called Reliability Deviation or RD. Glicko has 95% confidence that the player's true skill rating is within two standard deviations from their base rating. For example if a player's rating is 1500 (this script defaults starting player Glicko ratings at 1500), the algorithm is 95% confident that the player is skilled somewhere between 1400 and 1600. The RD of a player starts off quite high as the algorithm is not certain on a player's skill if they have only played a few matches. Gradually as they play more the RD shrinks which signifies a more accurate rating.  
The RD of a player affects the impact they have on the rating of their opponent. When the player's RD is low, it means that Glicko is relatively confident in their skill rating being accurate, which means that they will have a larger impact on the rating of their opponent and vice versa for when there is low confidence in a player's strength. 
Glicko-2 introduces a new variable called rating volatility which measures the degree of expecting fluctuation in a player's rating. This accounts for unexpected performances. This value is low when a player consistently performs at the level near where Glicko would expect and is high otherwise. Player volatility defaults to a value of 0.06 at the start.
### GXE
 - https://www.smogon.com/smog/issue43/elo-hello  
Glicko X-Act Estimate or GXE is a percentage score devised by members of the Competitive Pokemon Community that uses a player's Glicko rating to calculate their expected chances of beating any random tracked player. For example, if a player has a GXE of 60%, that means that they are expected to have a 60% chance of beating a random opponent. Players with GXEs that are lower than their actual win percentages means that they are overperforming based off their expected skill level against the rest of the playing field. Since we randomize our draft seating through a totally random system (Smash AI battles) rather than using Glicko to "matchmake" like in most online games, it can mean that a player is consistently being paired up vs opponents that are lower skilled than the average player. The opposite is the case for those with a GXE higher than their actual winrate. It means that they are consistently being matched up vs players that have higher skill level than the average random player.