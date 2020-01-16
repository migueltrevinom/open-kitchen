# Wok Simulation

This Wok simulation contains two parts:

1. **WokSim**: the core hardware level - single wok simulation
2. **WokI2CSim**: the Wok(s) simulation on the simulated I2C interface



# Virtual Environment Setup

You will need to setup a virtual environment to run the wok simulation. The instructions, of how to setup a python
 virtual environment using virtualenv, are the following,
 
```bash
virtualenv <environment_name>           # Create virtual environment
source ./<environment_name>/bin/active  # Activate virtual environment
pip install -r requirements.txt         # Install requirement at runtime
pip install -r requirements-dev.txt     # Install requirement for development
```

The `requirements.txt` and `requirements-dev.txt` are stored in the main root directory of this repository.

Once the virtual environment has been setup, go to the directory `Modules/WokModule/Simulation` in this repository so you are
 ready to run the Wok simulation.

# Wok Behavior

The WokSim is the core of hardware-level, single wok simulation. It simulates the behavior of a wok which will only
 receive I2C requests from the main controller (Raspberry Pi) and respond base on different requests from the main
  controller. The overall workflow cycle of the Wok component will be,

1. The Wok will initialize in the `waiting order` state, which will
    - Wait for the main controller to set the cooking heat temperature.
    - Wait for the main controller to set cooking duration in seconds.
    - Wait for the main controller to pass down the order id.
2. After the above parameters are set, the Wok goes into the `waiting ingredients` state which will
    - Wait for the main controller to notify if the ingredients are in the Wok.
3. Once the main controller confirm the ingredients have been put in the Wok, it goes into the `cooking` state which will
    - Heat the Wok to the temperature configured and cook for the duration in seconds previously set as well.
4. Once the cooking time has passed, the Wok will enter the `exporting dish` state which will
    - Drop the Wok content (dish) into the bowl.
5. Finally, the Wok will go to the `cleaning` state after dropping the dish into a bowl. It will 
    - Clean itself
6. The Wok will cycle back to the first step

The Wok also supports changes to the cooking time and temperature in the middle of the `cooking` state, if so, it will reset the Wok.



# WokSim CLI
 
WokSim can be executed in the terminal with the following command ( you have to be in the same directory as the `WokSim.py` ),

```bash
python WokSim.py
```

After you run the previous command, you should see something like the following screenshot,
![](../../../docs/img/wok_sim_001.png)

Now, you are can send requests as the main controller does to control the Wok. Every time you send a
 request, the Wok should respond with a code number. The following table shows what are all the request types that can be done and the possible responses:
  
#### Main controller to Wok 
| Request code | Request Description | Data  | Response | Response Description
|:-------------|:--------------------|:------|:---------|:--------------------
| 1            | Get component code  |       | 1        | Wok component
|              |                     |       | 2        | Ingredient dispenser component
|              |                     |       | 3        | Sauce runner component
|              |                     |       | 4        | Transporter component
| 2            | Get state code      |       | 1        | Waiting order state
|              |                     |       | 2        | Waiting ingredient stat
|              |                     |       | 3        | Cooking state
|              |                     |       | 4        | Exporting dish state
|              |                     |       | 5        | Cleaning state
| 3            | Get error code      |       | 0        | No errors
|              |                     |       | 1        | Error: cooking terminates before cooking time complete
|              |                     |       | 2        | Error: the bowl is not in place for dish exporting
| 4            | Get Wok request code|       | 0        | Wok no request
|              |                     |       | 1        | Wok request to set heat degrees
|              |                     |       | 2        | Wok request to set cooking duration in seconds
|              |                     |       | 3        | Wok request to set order id
|              |                     |       | 4        | Wok request to notify if ingredients is ready (been put in the Wok)
| 5            | Respond Wok request | uint8 | 0        | Wok failed to save data and/or not able to setup
|              |                     |       | 1        | Wok successfully save data and/or setup
| 6            | Reset Wok           |       | 0        | Wok failed to reset
|              |                     |       | 1        | Wok successfully reset
| 7            | Reset cooking time  | uint8 | 0        | Wok failed to reset cooking duration
|              |                     |       | 1        | Wok successfully reset cooking duration


The following table represents the request types from the Wok to the main controller ( Raspberry pi ) and the meanings of each one of them.

#### Wok to the main controller
| Request code | Request Description                     | Data  | Response | Response Description
|:-------------|:----------------------------------------|:------|:---------|:--------------------
| 0            | No request                              | 
| 1            | Request to set heat degrees             |       | uint8    | Heat degrees
| 2            | Request to set cooking duration         |       | uint8    | Cooking duration in seconds
| 3            | Request to set order id                 |       | string   | Order id
| 4            | Request to confirm if ingredients ready |       | 0        | Denied 
|              |                                         |       | 1        | Confirm

Therefore, we can use the I2C request code `4` to check the Wok request code. So, if you type 4 in the WokSim command line and hit
 enter, you should see the following,
 ![](../../../docs/img/wok_sim_002.png)
 
 As you can see, the WokSim response is `1`, which means the Wok is requesting us to set the heat degrees. We can use
  I2C request code `5` to respond Wok the heat degrees. You will notice that the command line asks you to enter the
   I2C data if we use I2C request code `5` which will allow you to enter the Wok heating degrees in Celsius. I will
    enter 100 as an example here, so you should see following,
 ![](../../../docs/img/wok_sim_003.png)
 
 Since the WokSim respond `1` to me after I set up the heating degree, so it has successfully setup the heating degrees
 . Now if you check the Wok request by using I2C request code `4`, you should find that the response changed to `2` which is
  means that the Wok is requesting us to set the cooking duration in seconds.
 ![](../../../docs/img/wok_sim_004.png)
 
 You can repeat the same steps as before but now we need to do the duration and the order id, to do that, we need to send the request code `5` to setup the cooking duration and order id. After you finish setting up those
  three parameters, the Wok should enter `waiting ingredients` state and you should see the screen below,
 ![](../../../docs/img/wok_sim_005.png)
 
 Now, the Wok will remain in a waiting state untill you confirm that the ingredients are in the Wok. You can confirm that using request code `5
 ` and enter the data = `1`. Then, you should see WokSim moves to the `cooking` state. After the cooking time is up, it will go
  through the `exporting dish` state and `cleaning` state and that's the last state in the loop, which means that it will go back to `waiting order` state automatically. The screen that you will see should be similar as the one below
 ![](../../../docs/img/wok_sim_006.png)
 
 Now, you can go over the same process to simulate another dish repeating all the steps above. 
 
 Note: You can type in `stop` in the I2C simulation terminal to exit the WokSim process.
  
  
# WokI2CSim

WokI2CSim is an extension of the WokSim which uses a RESTful API to simulate the I2C communication that offers a communicate with multiple Woks. WokI2CSim will only simulate 2 Woks at this point. 
 
To start it up, you need to go to the same directory where the file `WokI2CSim.py` is located and run the following command,
```bash
uvicorn WokI2CSim:i2c_sim
```

You should see something like this when it's up and running,
 ![](../../../docs/img/wok_i2c_sim_001.png)
 
The RESTful API will open at `127.0.0.1:8000` by default if the port is available and the user interface should be available if you visit `http://127.0.0.1:8000/docs`. 
 ![](../../../docs/img/wok_i2c_sim_002.png)
 
Then, you should able to control the Wok using the same process explained above (WokSim). You can select `1` or `2` as the `wok_id`. Also, you must put `0` in the data field if a request doesn't require to input any data.
 ![](../../../docs/img/wok_i2c_sim_003.png)
 
Each response in this RESTful API simulation will contain helper fields that translates the raw request and the raw response into a more human friendly message to understand what is going on easily. For
 example: 
 
 ```json
{
  "wok_id": 1,
  "raw_request": 4,
  "raw_response": 1,
  "request_description": "request wok action",
  "response_description": "The Wok request to set cooking temperature."
}
```

 
Here is what you should see if you did setup the Wok #1 to cook at the temperature of 100 C for 10 seconds.
 ![](../../../docs/img/wok_i2c_sim_004.png)
  
  
# The End
That's how you can use Wok simulation.