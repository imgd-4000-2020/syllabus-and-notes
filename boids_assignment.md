
# IMGD4000: Agent-based simulation assignment
There are three goals of this assignment:

1. Implement the [Boids algorithm](http://www.kfish.org/boids/pseudocode.html), originally presented [in a classic paper that has been cited over 11000 times](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=3&cad=rja&uact=8&ved=2ahUKEwiqy5OP_tPoAhWDuJ4KHdqmDH4QFjACegQIIhAB&url=http%3A%2F%2Fwww.cs.toronto.edu%2F~dt%2Fsiggraph97-course%2Fcwr87%2F&usg=AOvVaw2StOMrXs0E_nHLgD87UrGN).
2. Make a (very simple) game in Unreal. Everyone should be able to solo prototype simple games in UE4 after leaving this class.
3. Experiment with some extra game design / development work.

[Here’s a video of the original simulation in action](https://www.youtube.com/watch?v=86iQiV3-3IA). For an in-depth treatment of agent-based steering algorithms more generally (including Boids) try [this chapter from the Nature of Code](https://natureofcode.com/book/chapter-6-autonomous-agents/).  The author of this chapter also walks through implementing the algorithm (in 2D) in [his amazing Coding Train series](https://www.youtube.com/watch?v=mhjuuHl6qHM). But the first link in this assignment is perhaps the easiest to follow, with clear pseudocode and explanations of the algorithm. 

## Requirements
1. Implement Boids, preferably primarily using C++ (you can use Blueprints if you like, but must use C++ for the next assignment). 
2. Create a game using Boids. Simple ideas:
	1. Avoid the swarm — dodge collisions in a bounded space
	2. Destroy the swarm — shoot swarm agents to increase score. Respawn agents as needed
	3. Control the swarm — give players a UI to control parameters of the Boids algorithm.
	4. ??? Your idea goes here
3. Create a gameplay video and post it.
4. Post your repo to GitHub. Include a README that describes your game and links to your gameplay video.
5. DM me the repo link in Slack by Monday, April 20th

## Grading
60% : is boids implemented correctly   
20% : additional gameplay elements  
20% : correctly followed instructions (posted video, made GitHub repo etc.)  
