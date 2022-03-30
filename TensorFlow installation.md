# Google Coral TensorFlow installation ussing a Windows host

## System requirements
* A host computer running Linux (recommended), Mac, or Windows 10 
* Python 3 installed [check versions](https://www.tensorflow.org/install)

*	One microSD card with at least 8 GB capacity
*	Wi-Fi connection
 <br>

Since this is a windows installation [gitbash](https://www.stanleyulili.com/git/how-to-install-git-bash-on-windows/) terminal is recommended. However, before starting in gitbash python access needs to be provided for git bash.
```bash
echo "alias python3='winpty python3.exe'" >> ~/.bash_profile
```

```bash
source ~/.bash_profile
```

## [Flashing the Board](https://coral.ai/docs/dev-board/get-started/#install-mdt)<sup>[1]</sup>
I cannot stress this enough, but a freshly flashed board will be so much better when starting a new project*.
1.	[Download and unzip the SD card image](https://mendel-linux.org/images/enterprise/eagle/enterprise-eagle-flashcard-20211117215217.zip) 
The ZIP contains one file named flashcard_arm64.img.
2.	Flash the SD card using [balenaEtcher](https://www.balena.io/etcher/)
3.	To flash the device it needs to be set to boot from sd card mode, to do this [flip the switches](https://coral.ai/static/docs/images/devboard/devboard-bootmode-sdcard.jpg) as the figure (Power should be disconnected until this point)
4.	After inserting the card connect DC power the fan will turn on and the power led will light up the flashing takes about 10 minutes the power LED will shut down after the process is done the fan does not start to again spin until the temperature threshold (default 60 C is reached so it’s not malfunctioning).
5.	[Flip the switches](https://coral.ai/static/docs/images/devboard/devboard-bootmode-emmc.jpg) to position as shown in figure
<br>

## Mendel Development tools
MDT is a command line tool that helps you interact with the Dev Board however this is sometimes really hard to work with especially on windows
Installing MDT to your host computer.
*	In your gitbash terminal 
``` python
python3 -m pip install --user mendel-development-tool 
```

**important notice** 

*	MDT is not added to path, and it will mention so in the installation as well you need to add MDT to your path, note the installation location copy it and us the following command on gitbash
Mine was something like this for the path
```
C:\Users\chira\AppData\Local\Packages\PythonSoftwareFoundation.Python.3.10_qbz5n2kfra8p0\LocalCache\local-packages\Python310\Scripts
```
```bash
$ export PATH="$PATH:/c/users/chira/AppData/Local/Packages/PythonSoftwareFoundation.Python.3.10_qbz5n2kfra8p0/LocalCache/local-packages/Python310/Scripts"
```
*	We need an alias for mdt as well since we’re running windows.
echo "alias mdt='winpty mdt'" >> ~/.bash_profile

restart bash profile with

```bash
source ~/.bash_profile
```

## First time setup and internet connectivity
Now that mdt is properly functioning we connect the Dev Board to the host pc via usb c cable (otg port in dev board to pc usb C (or A)
To figure out if the connection and the board is functioning
mdt devices
will return
```
elusive-zebra           (192.168.101.2)
```

this hostname is randomly generated and the ip address is also provided.
For direct shell access to Dev board (If this doesn’t work refer troubleshooting section)
mdt shell

Now with access to the device we need to establish internet connection. For this case 
```bash
nmcli dev wifi connect <NETWORK_NAME> password <PASSWORD> ifname wlan0
```
One can also use the
 ```bash 
gui nmtui
 ```
 option but gitbash but this can result in the following error

**Your terminal lacks the ability to clear the screen or position the cursor.** 

I suggest the cli over gui.
To check the connection

```bash
nmcli connection show
```
Updating the Mendel Software 
With the internet connection established you can update the mendel software 
``` bash
sudo apt-get update
sudo apt-get dist-upgrade
```
now that this is done the coral is ready to run tensorflow lite models on the coral edge tpu. 
Important**
Before you do that I highly suggest using any other terminal and remote SSH and never use gitbash again.
* After flashing the hostname and password defaults to **mendel**
How this works is discussed further,
## TensorFlow Lite Installation
You are forced to use tflite instead of the full library as the edgt TPU only supports TF lite models 

```python
python3 -m pip install tflite-runtime
```

A slight modification is required for the standard tf code
```python
import tflite_runtime.interpreter as tflite
#instead of
import tensorflow as tf
```

and the interpreter is also changed
```python
interpreter = tf.lite.Interpreter(model_path=args.model_file)
```
The Edge TPU is capable of executing deep feed-forward neural networks such as convolutional neural networks (CNN). It supports only TensorFlow Lite models that are fully 8-bit quantized and then compiled specifically for the Edge TPU.
By default, the TensorFlow Lite interpreter executes your model on the CPU, but that fails if your model is compiled for the Edge TPU because an Edge TPU model includes a custom Edge TPU operator. So you need to specify a delegate for the Edge TPU. Then, whenever the interpreter encounters the Edge TPU custom operator, it sends that operation to the Edge TPU. As such, inferencing on the Edge TPU requires only the TensorFlow Lite API.
So if you already have code that runs a TensorFlow Lite model, you can update it to run your model on the Edge TPU by following the steps below to update your TensorFlow Lite code.
1)	Install the PyCoral API. 
```python
sudo apt-get update
   sudo apt-get install python3-pycoral
   ```
2)	In your Python code, import the pycoral
3)	 Initialize the TensorFlow Lite Interpreter for the Edge TPU by calling `make_interpreter()`
4)	Then use our `"model adapter"` functions to simplify your code—such as using classify.`set_input() and classify.get_output()` to process the input and output tensors—in combination with calls to the TensorFlow Lite Interpreter, as shown in the following example.

References

1. [Dev board wiki](https://coral.ai/docs/dev-board/get-started/#install-mdt)
2. [Tensorflow Lite guide](https://coral.ai/docs/dev-board/get-started/#install-mdt)
3. [Tensorflow installation](https://www.tensorflow.org/install)
