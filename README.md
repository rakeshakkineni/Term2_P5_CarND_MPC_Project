## **CarND Model Predicitive Control Project**

The goals / steps of this project are the following:
* Modify the code to make the car in the simulator drive autonomously without crossing the road limits for atleast one lap using Model Predicitive Control method.


[//]: # (Image References)

[image1]: ./Output/CTE_Vs_Time.png
[image2]: ./Output/Stering_Angle_Vs_Time.png 
[image3]: ./Output/Throttle_Vs_Time.png
[image4]: ./Output/Velocity_Vs_Time.png

---
### Writeup / README
In this project vehicle in the simulator is controlled using MPC Controller, the main task is to implement the MPC controller and tune the parameters such that the vehcile does not cross the road limits. Source code provided by "UDACITY CarND MPC Project" was used as base for this project. 

### Modifications
main.cpp , mpc.cpp are modified to implement MPC controller. FG_eval, operator, Solve , main() functions were modified to implement the controller . Modified code can be found [here] ("./src")

### Code Flow
Following is brief description of the flow of the code.
- Establish the connection with simulator.
- Read the vehicle state and way points from websocket for one scan.
- Convert Vehicle speed to Kmph from Miles Per Hour.
- Predict the vehicle state after 100mSec.
- Transform the way points to vehicle coordinates using the predicted state.
- Fit curve and get the coefficients of the curve. 
- Calculate CTE and PSI Error using the calculated coefficients.
- Pass Vehicle State , CTE , PSI Error to MPC Solver. 
- Send the steer_value and throttle value to Simulator.

Following is brief description of the flow of the MPC Solver.
 - Initialize the constraints for control and vehicle states.
 - Calculate the cost for control and vehicle states using operator. 
 - Return the solution (Steering and Throttle for first point and x,y positions for all the points) to main function 


### Code Implementation and Tuning Process
Following is the code developement flow for this project. 

- I have started with solution provided in "Mind the Line" quiz. I was succesful the drive the car autonomously on a straight road with minor changes to the cost calculation , however car was uncontrollable at turns. 
- As a second step i have changed rank of the polynomial that is being fitted for the way points. After this change vehicle was not even able to drive on the straight roads. After several trials i could not resolve the issues. 
- I have referred the following post on UDACITY forum https://discussions.udacity.com/t/attn-read-before-beginning-project/492944/7. I have followed the suggestions in the post (i.e. to convert vehicle speed to kmph and to transform the coordinates). I was partially succesfull after this change. However vehicle movement was wavey, i have checked the project walk through videos and editted my code inline with the walk through. After this vehicle was able to complete the lap at max speed of 60MPH.
- I was succesful in making the vehicle with an actuation delay of 100msec without prediciting the future state of the vehicle , however since it was a requirement of the project i have predicited the future state of the vehicle after 100msec using kinematic vehicle model. After this change vehicle movement was very unstable. I have changed the fg calculation method many times but i was not succesful in making the vehicle movement stable.

Tuning Steps:
- In tuning process primarily the factors multiplied to control parameter cost calculations were modified. 
- This was a painstakingly long process, to make things easier i have started priting out control parameters steer_value, throttle and vehicle speed, cte values to a file. After every run these values were plotted in a graph. If any of the parameters were observed to be oscillating or staturating then the factors multiplied to diff or current value are increased. 
- Following the above method i was able to make the vehicle movement stable however the vehicle speed was limited to 15MPH as throttle was severally damped. If the throttle was not damped vehicle became unstable if cost just included mean square of diff and actual value, equations shown below. 

  fg[0] += 2500*CppAD::pow(vars[delta_start + t], 2);
  fg[0] += 200*CppAD::pow(vars[a_start + t], 2);
  
  fg[0] += 3000*CppAD::pow(vars[delta_start + t + 1] - vars[delta_start + t], 2);
		fg[0] += 500*CppAD::pow(vars[a_start + t + 1] - vars[a_start + t], 2);


To increase the speed i have reduced the damping factor and have added fourth power diff cost parameter as shown below. 
  fg[0] += 2500*CppAD::pow(vars[delta_start + t], 2);
  fg[0] += 200*CppAD::pow(vars[a_start + t], 2);
  
  fg[0] += 3000*CppAD::pow(vars[delta_start + t + 1] - vars[delta_start + t], 2);
		fg[0] += 500*CppAD::pow(vars[a_start + t + 1] - vars[a_start + t], 2);

  fg[0] += 5000*CppAD::pow(vars[delta_start + t + 1] - vars[delta_start + t], 4);
		fg[0] += 500*CppAD::pow(vars[a_start + t + 1] - vars[a_start + t], 4);

After the above change speed increased to a maximum of 23MPH and vehicle was stable. I could not increase the vehicle with future state prediction any further. 

### Output
Following picutres the trend of Angle , Steering Value and CTE over time. 

- CTE Vs Time 

![alt text][image1]

- Steering Value Vs Time

![alt text][image2]

- Throttle Vs Time 

![alt text][image3]


- Vehicle Speed (KMPH) Vs Time 

![alt text][image4]


- Simulator and Command Prompt Recording can be found [here]("./Output/Term2_P5_Output.mp4").


## Result 
- MPC was tuned and vehicle movement was quite stable.
- Vehicle was above to achieve a maximum speed of 23MPH (37KMPH).
- Vehicle takes around 1Min 40Sec to finish a lap.
