# Bag runner optimization in an airport operation
A heuristic goal programming, binary LP built using Gurobi/Python that picks best gate route from pareto solution pool to reduce flight delay due to bags delivery at finite number of departure gates by a baggage handling vehicle which tows finite number of carts.

## Please view the code file for detailed report

**Background**:
The airline management often requires sophisticated and complex mathematical models to operate even the most mundane tasks that could be executed with very less thought as it results in a huge cost savings during the operation. One such operations is carried out by a Rampie. A Rampie is the driver of a baggage runner vehicle that tows variable number of carts depending on the airline departure schedule and traffic, which delivers baggage to its corresponding airline gate from the central bag repository. The bags of the passengers after being handed over at the check-in counter reach via an automated conveyor system a specified spot in the central baggage room from where they get picked by the rampie.

**Challenge**: The rampie often faces a situation where there would be multiple flights waiting for baggage and they would be having departure deadlines very close to each other and choosing between which baggage to pick in to cart first and first stop of delivery is extremely sensitive and even a slight deviation from the optimal delivery would lead to huge delays in flight departures and adding to it the irony of having no clue about what the optimal solution is in the first place. Designing an Operations Research mathematical model to find this optimal delivery sequence is very demanding in the airline industry and also very challenging.

In an airline industry, passenger experience and feedback play a very crucial role in its potential to dominate the market. In this exercise a very similar problem statement has been setup in a much simpler way and a solution approach has been proposed.

**Problem statement**: Considering an airport operation where the airline company has been allotted gates 1 to 9 to operate their departure business. Flights ready to take off from all these 9 gates are waiting for their baggage to get delivered by the rampie. The departure times of these flights are scheduled as shown in the below table. The rampie could tow 4 carts at a time and each cart has enough space to load baggage of a single gate. No cart can be loaded with baggage belonging to multiple gates but only a single gate. For simplicity assuming the central baggage room is located at the very beginning of the gate line and each gate is located in a linear layout with equal distances from their neighboring gates as depicted in picture below. Assuming loading and unloading time is 5 minutes and 2 minute to travel from a gate to its neighboring gate, the goal is to find out the best sequence of delivery that minimizes delay in gates due to delay in delivery (if any).

![](Images/Schedule.png)
![](Images/Gates_layout.png)


**Result format**
As this repository is being public, I would like to conceal the reasoning,logic of the constraint developement, however, the mathematical form is shown in code.

The final result is produced in a tabular format as shown in the below example. Starting from the left, "attempt 0" describes the initial move/action taken by the rampie, indeed picks(p) bags belonging to gates-(2,5,8,9) which happens only at gate-(0)/central baggage room and the next move "attempt 1" is made to drop(d) baggage belonging to gate-(2) and the next one dropped(d) at gate-(8). "attempt 3" again shows the rampie moving to central baggage room to pick(p) baggages of gates-(6,7).The final attempt/move made was during "attempt 12" where baggage to gate-(4) has been delivered. 0d indicates that those atempts were not used and final delivery was accomplished before reaching to that attempt

"total_time" indicates how many hours it took for the rampie to finish delivery to all gates and "sequence" shows the same route performance in a string format.

![](Images/Result.png)

**Heuristic optimization approach**
The objective of the model is to make sure that all gates receive their baggage on time or with least delay possible. It would be much easier to write an objective function that minimizes the total working time of the ramper vehicle, but it only favors the rampie to leave work and go home early rather than focusing on delays caused at the gates. To be precise, there would exist two different paths that have the same total travel time but only one of those paths might be able to deliver baggage on time at all gates while the other might not. When trying to minimize total delays at each gate, there would be a trade off in delivery delays within gates. In that case it would be difficult to choose one over the other as we have no specific goal to look for, hence named this model a goal programming model.

A heuristic approach using Pareto feasible solution pool The usual way of solving a goal programming formulation is by setting weights/penalties to different objectives and concatenate into one final objective function, however, I have decided to approach this problem in an unusual way.

Pareto optimal solutions are those solutions within the decision space whose corresponding variables cannot be all simultaneously improved. I would first print out all possible combinations of solutions using Gurobi just based on constraints; and by setting a dummy objective function in the code. This would give me a pool of solutions that the rampie could choose from and out of this pool of solutions I would filter out for those solutions that have no delay at any of the gates. If such solutions exist, I will then choose the one with least total job time. If such solutions do not exist, it means that a perfect delivery with no delay at any of the gates is impossible for that situation. If there exist only solutions of such kind in the pool, the best way of approach that one might think is to choose the one having least number of gates delivered with delay (each with some amount of delay) and among them again choose the one with least total job time.

However, choosing the solution with least number of delayed gates poses a caveat. Suppose if we pick a solution with only one gate say gate-6 delayed by an hour and others delivered on time, one will rather wish for a solution that observes a 10-minute delay at 5 gates which compensates for gate-6 delay, i.e., just a 10 min delay at gate-6 as well. In order to promote such compensations, I will choose a delay threshold allowing no gate to delay beyond that threshold and choose among the solutions filtered based on this threshold value, the one with least number of gates delivered with a delay. If failed to find a solution, I would then increase the threshold by some small value (say 30 seconds) and repeat the process again until a solution has been found.

A final way of approach to choosing the solution is explained in the pseudo flow chart as shown below.

![](Images/Flow_chart.png)

**A major flaw during the execution**
In this Jupyter notebook the depicted algorithm has not been put to use and in fact, no solution has been produced due to having found a major flaw with the solution pool. Hence, the solution selection has been halted for now and still working on resolving the issue.

**The flaw:**
Not every possible combination of feasible solution is produced in the solution pool.
Expectation: Due to the objective function being a dummy value, Gurobi has no final value to look for and Gurobi’s branching has been progressed only with certain combination of variable assignments and produced every possible feasible solution into the pool with those certain combinations. In fact, when tested a missing solution by assigning it as a constraint, the solver did not show an infeasibility status, which proves that the missing solution is a valid feasible solution and should not have been missed in the pool.

