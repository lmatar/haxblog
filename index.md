![Haxball!](/images/haxballtitle.png)

# Introduction
Try to implement some of the creative intros

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
`  
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
    #print("Calculate intersection at time: {}".format(kick["time"] - offset))
    #frame = get_positions_at_time(match["positions"], kick["time"] - offset)
    #Using in range and tracing back to see what frame was right before it left the foot
    #A list of lists with only info about player we want and ball
    shooter_frames = []
    ball_frames = []
    #print(kick['fromName'])
    for i in frame:
        if i['name'] == kick['fromName']: 
            shooter_frames.append(i)  
        elif i['type'] == 'ball':
            ball_frames.append(i)
    #print(shooter_frames)
    #print(ball_frames)
    #picking frame with least dist
    least_dist = float('inf')
    player_position = {}
    ball_position = {}
    #List of dicts
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
    #print(frame)
    #print(player_position)
    #print(ball_position)
    #Getting goal positions
    goal_mid = get_opposing_goalpost(stadium, kick['fromTeam'])['mid']
    #print(goal_mid)
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
    `
    
### Player speed
### Weighted Defender Distance

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

## Special thank you
Thank you to Vinesh Kannan for getting all this data and giving me the opportunity to work on the project and learn so much!
