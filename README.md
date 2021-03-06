# Evolving recurrent neural networks for reinforcement learning and adaptive foraging


## Task description

An agent has access to two kinds of randomly-moving "preys". One kind of prey is food, while the
other kind is poisonous and must be avoided. However, the agent does not know
which is which. Furthermore, the prey types switch during each episode (food
becomes poison and vice versa).

![Bugs UI](https://github.com/ThomasMiconi/Bugs/blob/master/World.gif)

The agent is controlled by a simple recurrent neural network, with two
actuators (speed and heading) and six sensors (one for each type of prey on the
left and right, one for 'pain' and one for 'yum'). It can detect each prey type
on either side of its body (with intensity inversely proportional to distance),
and can  also detect whether whatever it just ate is food ('yum') or poison
('pain'). From this, it must learn the relevant associations, find out which prey type is food
and which is poison, and adapt when they switch.

## Evolution

The neural network that controls the agent is designed by evolution.
The evolutionary algorithm is a simple real-valued genetic algorithm (inspired
by Karl Sims): Evaluate each agent in turn by letting them interact with the
environment for 10,000 timesteps and calculate their total score (simply the
total number of food minus poison eaten). When the entire population has been
evaluated, select the 20% best, and replenish the rest of the population with
mutated copies of these 20%. Mutation occurs by adding a Cauchy-distributed
number to 5% of the network's weights. There is no crossover and parents for
each new individual are chosen randomly from the 20%, in order to maximize
exploration. In addition, we disallow direct connections
from input sensors to actuators. This forces the sensors to take input from
other neurons, which seems to facilitate the discovery of the relevant
computations.

## Results

It turns out that there are two main strategies to solve this problem: a workable, but suboptimal strategy, and an optimal strategy. 

The suboptimal
strategy is to follow one type of prey, and if too much pain is detected,
abandon any chasing and simply spin around; while spinning (since you will randomly catch some preys), if a lot of 'yum' is detected,
resume the chasing of the preferred prey type.

This strategy works because while spinning, you get random (and thus roughly
equal) amounts of poison and food, while when chasing you will get a lot more
preys of your preferred type. The pain/'yum' detectors can tell you whether it
is currently better to chase or spin. Importantly, you don't need to switch
your preferred prey - just whether you chase it or not. As a result, this
strategy is simple, and evolves readily in every run.

The optimal strategy is simply to switch the type of prey you are chasing
depending on pain/'yum' inputs. This strategy yields much higher scores, but is
more complex to implement (it requires a true 'cognitive switch' rather than
simply a brake on motor outputs) and thus harder to evolve.

## How to run

You need a Java interpreter. To observe a pre-evolved agent, just run `java
World FILENAME bestagent_default.txt`. This will start the GUI program and only
display this saved agent, without
any evolution going on. You can 'switch' which type of prey is poison or food by
clicking the button - and see how quickly the agent adapts.

To run the actual genetic algorithm and evolve a new agent, simply run  `java
World`.  This will start the genetic algorithm but will not produce any
graphical output. After each generation, the current best agent will be stored
in a file called 'bestagentXXX.txt' (where 'XXX' is a bunch of parameter
values). To visualize this agent, run `java World FILENAME bestagentXXX.txt`.
This will start the graphical interface and allow you to observe the agent's
behavior.  You can press the + and - buttons to increase or decrease the
refreshing time (reduce refreshing time to 0 for maximum speed). With default
parameters, the algorithm starts to increase performance by ~450 generations
and produces a near-optimal strategy within ~1400 generations. 

The code is written in simple Java to encourage tinkering. Fire up your
editor and code away!

To recompile the Java bytecode, simply run `javac World.java`. If
you modify any other Java source file than World.java, you need to recompile
them explicitly too (or alternatively, just run `rm *.class; javac World.java`
to force a full rebuild). 



## Single-agent vs. populations

There are two branches in this repository. The main branch (`master`) evolves
single agents. The other branch (`Populations`) evolves "swarms" of agents,
called Populations (yes, this is confusing). By default, those populations only
contain one agent and are entirely equivalent to single-agent evolution.
However, they provide the infrastructure for future experiments in evolving
collaborative multi-agent systems.


## Related work

It is well-known that suitably trained recurrent networks can perform
reinforcement learning - that is, learning from sparse, delayed rewards (for
recent work on this, see [Wang et al.  2017](https://arxiv.org/abs/1611.05763),
[Duan et al.  2016](https://arxiv.org/abs/1611.02779)). Evolving recurrent
networks to do this has a long history (see [Blynel and Floreano
2003](https://link.springer.com/chapter/10.1007/3-540-36605-9_54); [Yamauchi and
Beer 1994](http://dl.acm.org/citation.cfm?id=189951)). This code applies the idea to a foraging task for situated agents chasing mobile prey.
