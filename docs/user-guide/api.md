# API

Task definition files in pyControl run directly on the Micropython microcontroller, not on the computer, so the majority of python packages (for example Numpy and Matplotlib) cannot be integrated into task logic. Therefore an application programming interface (API) is provided which allows python running directly on the computer to communicate with the board while tasks are ongoing. This communication is two-way, whereby code run on the computer can both receive data from the board and alter task logic.

## Example Usage

Two examples of this functionality are provided in api/user_classes. 

### Blinker
This class interfaces with the task definition file blinker.py and demonstrates how to change a task variable while the task is running. The Blinker API class uses the inherited `set_variable` function to change the task variable `LED_n` every time the tasks transitions into the `LED_off` state.

### Reversal_learning
This class interfaces with the task definition file reversal_learning_api.py and demonstrates how to integrate a pyControl task with Matplotlib plotting. The Reversal_learning API class calls the inherited `setup_figure` function on initialisation to open a matplotlib figure. While the task is running, the `process_data_user` function reads the lines printed by the task and updates the plot based on the printed trial summary line.

## Writing your own API class

### The task file
Your task file must initialise a task variable `api_class` with its value a string as the name of your API class.

For example:

```python
v.api_class = 'Blinker'
```
### Basic structure of the API class
Write your API class in a new file in the directory api/user_classes. You must import the Api class from api.py and inherit from it 

*For example:*

```python
from api.api import Api

class Blinker(Api):
```

### Available functionality

The functionality available to your API class is outlined in api.py. The superclass provides functionality by assigning attributes to your class and through several functions which can be used in your class either by overwriting them or calling them. It is indicated both in api.py and in the Function Reference here which functions can be overwritten or called and which should not be interacted with by the user. Available attributes are listed in Variable Reference.

Your API class can overwrite some of the inherited functions. These functions are called at different points during the task, so if overwritten, your function will be called instead.

*For example this function:*

```python
def run_start(self):
    self.print_to_log('\nUsing api to plot reversal learning moving average of choices')
```
In the Reversal_learning API class overwrites the default `run_start` function in the Api class and causes a line to be printed to the log when the task is started.  See function reference for details about all functions that can be overwritten.

Other inherited api functions can be called directly from within your class. These should not be overwritten.

Finally there are several private functions to the Api class that will elicit undefined behaviour if overwritten or called. These are clearly indicated and are not included in the function reference.

## Function Reference 

### Functions to Overwrite

---

**run_start**

```python
run_start(self)
```
Called once when the task is started

---

**run_stop**

```python
run_stop(self)
```
Called once when the task is stopped

---

**process_data_user**

```python
process_data_user(self, data)
```

Called whenever there is a state transition, event
or printed line. Gives the user access to data dictionary

data : a dictionary with keys 'states', 'events' and 'prints'
       and values a list of tuples in format
       (name of state or event / printed string, time)

---

**update**

```python
update(self)
```
Called several times / second regardless of whether
there is a state transition or event.

The user should be cautious when overwriting this function
as code that does not execute in sub-millisecond time will
impact the performance of pyControl

---

### Callable functions

---

**set_variable**

```python
set_variable(self, v_name, v_value)
```

Call this function to change the value of a task
variable.

v_name : str,
name of variable to change 

v_value :
value to change variable to

---

**setup_figure**

```python
setup_figure(self)
```
Creates a figure for each setup and assigns figure 
and ax attributes to user class api. 
Sets the title of the figure to subject ID if called as
part of an experiment

## Variable Reference 

*Variables available to your api class through inheritance:*

`self.board` Pycboard object. Contains various attributes used for serial USB connection with the board. 

`self.print_to_log` Function. Call with a string argument to print that string to the txt file and GUI log.

`self.ID2name` Dict. Converts a event / state ID to its name string.

`self.figure` Matplotlib figure. Requires setup_figure to be called.

`self.ax` Matplotlib axis. Requires setup_figure to be called.

`self.experiment_info` Dict. Experiment only. Contains information about the experiment such as the task name and the task variables.

`self.setup_idx` : int. Experiment only. Index of the current setup.

`self.subject_id` : str. Experiment only. Id of the current setup's subject.

`self.other_APIs` List. Experiment only. A list of references to every setup's API class (including the current setup). 
Can be used to alter the behaviour of other setups based on the behaviour of the current setup.

