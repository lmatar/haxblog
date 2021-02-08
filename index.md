![Haxball!](/images/haxballtitle.png)

# Introduction
Before we start diving into the different aspects of this project, I have a challenge for you!

Let's look at two screenshots...

### Shot #1
![Haxball!](/images/intro2.png)

### Shot #2
![Haxball!](/images/intro1.png)

From the two pictures shown above, which shot is more likely to be an **expected goal** in your opinion?
* [ ] Shot #1
* [ ] Shot #2

Now lets look at the expected goal score that my model predicted...
#### Shot #1
Expected Goal = 0.193

#### Shot #2
Expected Goal = 0.510

*Note: If you are not sure what these values mean, don't worry! I will be explaining it soon.* 

Now with the images and the expected goal value, which shot is more likely to be an **expected goal** in your opinion?
 * [ ]  Shot #1
 * [ ]  Shot #2

Let's dive deeper into this project and see what expected goal (XG) is and how I was able to make a machine learning model to predict it!

*If you want to see if your guess was right, read until the end to find out!* 

# What is Haxball?
Explain what it is, urge them to play it, add some gameplay and screenshots!

# What is XG?
Explain the basics of XG (Add some visuals)

## Why not just use shots on goal instead of XG?
Explain why it is better!

# How do we predict XG?
Steps on how we are predicting it and what factors we looked at

# Haxball Machine Learning Project!
Now for the part you have been waiting for! 

## What data was collected?
Look at data exploration

## Which features were created?
List the features made and talk about the two/three main ones
### Shot intersect
``` 
def shot_intersection(match,kick, stadium, frame):
    '''Finds where the ball would intersect
    Args:
    match: Which match it is
    kick: Which kick we want to measure
    staduim: What stadium size it is (so we know where the goals and bounds are)

    Returns:
    Int 1 or 0 if the ball is going towars the goal or not
    '''
    #Getting last frame before the kick 
    #Using in range and tracing back to see what frame was right before it left the foot
    #A list of lists with only info about player we want and ball
    shooter_frames = []
    ball_frames = []
    for i in frame:
        if i['name'] == kick['fromName']: 
            shooter_frames.append(i)  
        elif i['type'] == 'ball':
            ball_frames.append(i)
    #picking frame with least dist
    least_dist = float('inf')
    player_position = {}
    ball_position = {}
    length = min(len(shooter_frames),len(ball_frames))
    set_dist = 30 #ball and player are at least get within 30 units then we assume that it was kicked
    
    for i in range(length-1,-1,-1): #frame len of ball and shooter should be the same
        dist = stadium_distance(shooter_frames[i]['x'],shooter_frames[i]['y'],ball_frames[i]['x'],ball_frames[i]['y'])
        if dist <= set_dist:
            player_position = shooter_frames[i]
            ball_position = ball_frames[i]
            break
    #If a frame isn't found with dist that is <= 30, then use least dist to calculate
    if not (ball_position) or not (player_position): #The dictionaries were not populated yet so default to getting least distance
        for i in range(length-1,-1,-1): #frame len of ball and shooter should be the same
            dist = stadium_distance(shooter_frames[i]['x'],shooter_frames[i]['y'],ball_frames[i]['x'],ball_frames[i]['y'])
            if dist <= least_dist:
                player_position = shooter_frames[i]
                ball_position = ball_frames[i]
                least_dist = dist
            else: #stopped decreasing
                break
    #Getting goal positions
    goal_mid = get_opposing_goalpost(stadium, kick['fromTeam'])['mid']
    #Extend line from shot angle (can't extend lines easily)
    if(len(player_position)==0 or len(ball_position)==0):
        return None, None, None
    y_val = point_slope(
        player_position,
        slope(player_position['x'], player_position['y'], ball_position['x'], ball_position['y']),
        goal_mid['x']
    )
    #Checking if the projection between the posts
    intersect = { 'x': goal_mid['x'], 'y': y_val }
    return player_position, ball_position, intersect
``` 
    
### Player speed

```
def speed_player(match,kick,player_name,positions):
    '''' Speed of the player
       Args:
           match: Which match it is
           kick: Which kick we want to measure
           player_name: What player do we want to measure the speed for

        Returns:
           Int that represents the speed of the player
    '''
    player_pos = []
    for i in positions:
        if i['name'] == player_name: 
            player_pos.append(i)    
    #Getting the time
    if len(player_pos) > 1:
        last = len(player_pos)-1#getting last index)
        time = player_pos[last]['time'] -  player_pos[0]['time'] 
        #Getting the distance 
        distance = stadium_distance(player_pos[0]['x'],player_pos[0]['y'],player_pos[last]['x'],player_pos[last]['y'])
        return distance/time
    else:
        return 0
 ```
### Weighted Defender Distance

```
def defender_feature_weighted(match,kick,stadium,positions,dist=0):
    '''Figuring out the closest defender and num of defenders for a kick
        Note: This is weighted so that defenders that are close to the player/ball or the goal count as 1.5 rather than 1
        Args:
            match: Which match it is
            kick: Which kick we want to measure
            dist: Set distance to consider a player pressuring

        Returns:
                List that contains the distance of the closest defender and the number of defenders (weighted)
'''
    closest_defender = float('inf')
    defenders_pressuring = 0
    ret = [0,0]
    for person in positions:
        if person['team'] is not kick['fromTeam'] and person['type'] == "player": 
            defender_dist = stadium_distance(kick['fromX'],kick['fromY'],person['x'],person['y'])
            if defender_dist < closest_defender:
                closest_defender = defender_dist
                ret[0] = closest_defender
            if defender_dist <= dist:
                #Checking distances  for weights
                post = get_opposing_goalpost(stadium, kick['fromTeam'])
                goal_dist = stadium_distance(post['mid']['x'], post['mid']['y'] ,person['x'],person['y'])
                if defender_dist <= 5:
                    defenders_pressuring += 1.5
                elif goal_dist <= 5:
                    defenders_pressuring += 1.5
                else:
                    defenders_pressuring += 1
                ret[1] = defenders_pressuring
    return ret
```

## What models were tested and used?
Talk about different types of models and their pros and cons.
Talk about top models and which one we chose (Random Forrest)

## Overall results and improvements
What the results were and how I improved from Edwins benchmarks 

# What were the main takeaways from the project?
Explain what I learned and what I liked and what I would change if I could

# How to view my model?
Steps to see the model
Link to the Github repo

### Contact Information

Email: lmatar@hawk.iit.edu

## Special Thank you to...
Vinesh Kannan for getting all this data and giving me the opportunity to work on the project and learn so much!
