# Electrical Vehicle Route Planning on Google Map with Reinforcement Learning
More datail please find the **```Paper.pdf```** experiment section. This work is aim to take battery, motor, traffic time, and other factor into account for generating better route for the vehicle under certain condition.

## Library Requirements with Python 3.6.2:
(1) Tensorflow (CPU)
(2) numpy
(3) pandas
(4) haversine
(5) time
(6) random
(7) math
(8) requests
(9) urllib


## How can you get start
Download all the python file as following: **```main.py```**, **```Environment.py```**, **```DoubleDQN.py```**, **```battery.py```**, **```motor.py```** <br />
(1) enter your start position: name of position or geocode (lat, lng)<br />
(2) enter your destination position: name of position or geocode (lat, lng)<br />
(3) enter the length of each step, higher but less accurate (ex: 1000m takes less time to train compare to 100m)<br />
(4) enter how many episode you want to train<br />
Make sure that you can access to the internet and google map during the training process. Notice that your server will be blocked if your access to google map API exceed the limit in 24 hour and your program will be stucked. In the **```main.py```** we have implemented a mechanism to make your program sleep for a while when you over query the data. If you can get the full access to the Google map API, then you can remove the ```sleep``` command and make the learning process faster.

## How can you be creative <br />
(1) You can model a real battery system which can include the battery degradation, SOC and other factor to make the whole system more like a real vehicle. The model can be implement in the python file: 
**```battery.py```**. In the original python file, we only model the battery in linear manner.<br />
(2) You can model a real motor system which can include the motor fatigue, heat condition and other factor to simulate the real motor. Your model can be implement in the python file:
**```motor.py```**. In the original python file, we only model the motor with idel manner. You can find out the real factor between input energy and output energy in the motor and apply in the code.<br />
(3) You can have a more complicated neural network architecture to deal with larger map boundary and implement in the file:
**```DoubleDQN.py```**. The function of this python file is to get input from the environment (such as state) and output an action (it can be Q values)<br />
(4) You can implement other learning algorithm other than Double-DQN in the file: **```main.py```**. This file will creat two .csv files , one record the training environment parameter names ```train_para``` and the other one names ```result``` will record the training data such as reward, duration, failed steps, step history and so on. The learning model and checkpoint is also saved in a folder names ```model```.<br />
(5) You can add more action choices in the function of ```step``` of file: **```Environment.py```** such as heading southwest or northeast. In the origine python file, we only implement 4 action: north, east, south and west. It is also possible to change the way you calculate the energy consumption and add the concept of regenerative brake.


## How does this work
### (1) Algorithm: Double-DQN<br />
<p align="center">
  <img src="/image/al.JPG" height="40%" width="40%">
</p>
By Google Deepmind (link:https://deepmind.com/research/publications/deep-reinforcement-learning-double-q-learning/)<br />

### (2) Learning Environment<br />
Make the map like a grid map for the learning agent to navigate on (figure(a)). Strictly speaking, each grid in the grid map is not a rectangle. This phenomenon is caused by the sphere geometry and our restriction on the length of the stride which is demonstrated in figure (b)<br />
<p align="center"><img src="/image/GridMap.JPG" height="60%" width="60%"></p>

### (3) Interact with Google map API <br />
assume that the agent is at current position denoted by A and heading south to the next position denoted by B. Apparently, the route provided by the Directions API is the highway 395 and the distance is longer than 1000m. We can get the navigating instruction list with the form: { geocode of A, duration from A to 1, distance from A to 1, geocode of 1 }, {geocode of 1, duration from 1 to 2, distance from 1 to 2, geocode of 2 }.... where A, 1, 2 ,3 , B are shown in figure 6 after we input the geocode of A and the geocode of B into the Directions API. The number of instruction is based on the Directions API and there are four instructions in our case of following figure from A to B. We use each of the geocode in the navigating instruction list to get the height of each position from the Elevation API and compute the elevation within each instruction.
<p align="center"><img src="/image/map1.JPG" height="60%" width="60%"></p><br />


### (4) Energy Computation<br /><br />
The following graph shows how to compute the energy for a vehicle to travel uphill (can also be applied on a flat road)<br />
<p align="center"><img src="/image/car2.JPG" height="60%" width="60%"></p><br />
But noticed that we only compute the elevation between the two position shown in the following graph. To increase the accuracy, you should minimize the distance between these two position (this will increase the computation time).<br />
<p align="center"><img src="/image/car1.JPG" height="60%" width="60%"></p><br />

### (5) Battery<br />
In the experiment, the battery performance will not affect the training process. The battery is able to carry totally 50000Wh of energy which is a standard offering by electrical vehicle manufacture, Tesla. In electrochemistry, it is recommended to use the the state of charge (SOC) from 90% ~ 20% of a battery to improve it's life which we implement in our case. The SOC is calculated by the ratio between the current energy and the total energy. We will not take the battery degradation into the experiment. Further work can take the real factor on battery performance into account as part of the training process. For this experiment, we only demonstrate how much energy consumed and how many times the battery need to be charged in an ideal condition. <br />

### (6) Reward arrangement <br />
The fundamental concept of defining the reward is based on the energy consumption in one stride from the current position to the next position, for example, from A to B shown in the figure 6. The energy is calculated by the method provided previous section. We then divide the energy by 10000 and times -1. In order to minimize the number of total steps during training, we add -0.1 to each transition if the next position is reachable. In other words, the reward r for taking any reachable step will be r = -0.1 - (energy consumption / 10000). If the next position is unreachable such as a lake or a river, r = -1 and the agent stay at the same current position and take the other action. If the distance of the next position and the destination position is less than the length of the predefined displacement (1000m in the figure 6), the reward r for taking this action will become r = +1 - (energy consumption from the current position to the next position / 10000) - (energy consumption from the next position to the destination position / 10000). Noticed here +1 appears in the reward because of the success of this action which lead the agent to the destination.<br />  
**Black:** Unreachable or out of boundary<br />  
**Red::** Start position<br />  
**Blue:** Destination position<br />  
**Grean:** Reachable position to reachable position where is not within the range of one stride of the destination<br />  
**White:** Reachable position to the position where is in the one stride length of range from the destination<br />   

<p align="center"><img src="/image/Reward.png" height="50%" width="50%"></p><br />


## Result
![result](https://user-images.githubusercontent.com/25232370/33046240-25c5ac2e-ce1e-11e7-9156-faf109c42bbe.JPG)<br />

## Credit
I learn a lot from Arthur Juliani's tutorial website which implement reinforcement learning algorithm with tensorflow. I also reference part of his code and modified it. (link: https://medium.com/emergent-future/simple-reinforcement-learning-with-tensorflow-part-0-q-learning-with-tables-and-neural-networks-d195264329d0)<br />
