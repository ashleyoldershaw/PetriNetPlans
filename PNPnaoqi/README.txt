==================================
   PNP library for Pepper robot
         Luca Iocchi 2017
      iocchi@dis.uniroma1.it
==================================

1. Install naoqi C++ SDK and create a qi workspace

   http://doc.aldebaran.com/2-5/dev/cpp/install_guide.html#cpp-install-guide


2. Compiling
	
	* Follow instructions to compile PNP in PNP folder

	* Compile PNPnaoqi stuff:

	$ cd <PATH_TO>/PetriNetPlans/PNPnaoqi

    For linux systems:

	$ qibuild configure -c linux64 -w <qi_ws>
	$ qibuild make -c linux64 -w <qi_ws>

    For Pepper robot:

	$ qibuild configure -c pepper -w <qi_ws>
	$ qibuild make -c pepper -w <qi_ws>


3. Preliminary Test

    * pnp_test
	- run 'pnp_test' in the folder containing 'plans' 
	  $ build-linux64/sdk/bin/pnp_test 
	  Program will terminate and you should not see any error.


3. Plan generation

    In the plans folder run

    $ ./gen_plan.bash <planname>.plan [<executionrules.er>]

    to generate <planname>.pnml

    Example:

    $ ./gen_plan.bash test3.plan test3.er



4. Plan execution

	
	* quick test
	- run 'run_test.sh' in the PNPnaoqi folder

	* full instructions
	- launch naoqi
	  $ <PATH_TO>/naoqi-sdk-2.5.5.5-linux64/naoqi
	- launch actions
	  (set PEPPER_IP=localhost if needed)
	  run the action servers (cd actions; python init_actions.py)
	- launch pnp_naoqi
	  run 'pnp_naoqi' in the folder containing 'plans'
	- send the plan to be executed to pnp_naoqi
	  run 'run_plan.py' with the name of a plan
	  $ python run_plan.py --plan <plan_name> (e.g., 'python run_plan.py --plan test1')
	  (actions are started and the plan terminates after execution, plan can be restarted 
      or stopped with 'python run_plan.py --plan stop')


5. Write actions

    An action requires a name and a procedure that implements it. There are two ways of defining actions: using Python functions (see example in action_A.py) or using a class (see example in action_B.py).
    The action procedure will be executed in a thread monitored by the do_run variable.
    Make sure that the action is reactive to this variable and that it stops as soon as do_run variable is set to False outside the action itself.

    Class-based actions can use self.do_run to read the action control variable and self.session and self.memory_service to access NAOqi session.
    Function-based actions can access such values through the getattr function.

    Each action file has to include an init(session) and a quit() function that are used to automatically initialize the action by a general action initialization script.


6. Test single actions

    Start the action servers and use action_cmd script to start/end/interrupt single actions

    Example:

    Terminal 1:
    $ cd <PetriNetPlans>/PNPnaoqi/actions
    $ python init_actions.py

    Terminal 2:

    & cd <PetriNetPlans>/PNPnaoqi/py
    python pnp_cmd_naoqi.py -a <actionname> -p <params> -c <start|end|interrupt>


7. Uploading and running on the robot

	On local machine: 
	edit 'upload.sh' to match your Pepper environment and run it

	OnPepper:
    execute actions and plans as described before
    (without launching naoqi and setting PEPPER_IP=localhost)	


==========================================================================

== Note about timeouts ==

There are two ways of introducing timeout of actions in the plan files.

1. Executing an action up to n seconds

Use this form in the plan file 'A|n'

Example:

  lookfor|30

will execute the action lookfor for at most 30 seconds, then it will continue with the next action.

2. Using the timeout condition

For each action an automatic timeout condition timeout_<actionname>_<time> is available.
An execution rule can thus contain rules based on this event.

Example:

Plan:
    lookfor

ER:
    *if* timeout_lookfor_30 *during* lookfor *do* somethingelse; ...

In this case after 30 seconds from the start of the acction, the condition timeout_lookfor_30
will be true and will activate the recovery procedure of the ER


