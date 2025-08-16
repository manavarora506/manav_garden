
What is windowing?

- Window input data into fixed size windows
- Process each of those windows separately
- Example:
	- Sum of all website clicks every hour
	- Average of food order total cost every hour
	- Max latency from a bunch of metrics observed every hour

How does it work?
- System essentially buffers up incoming data into windows until some amount of time has passed
- After that result will be sent downstream

How does state work?
- While the window is live all events are stored in memory
- once window has been finalized and message is sent downstream, you clear that window's data from memory 

What time to use for windowing?
- Processing time: you don't care about actual time when event happened
	- website logs
	- system metrics
- Event time
	- need a way to handle late data
	- in an unbounded stream you don't know when you are done processing events from a particular event time window
	- Events can arrive late and out of order
	- Example:
		- summing all events from 10AM-11AM
		- You sum 500 at 11 AM, but then you get another event. What do you do?
