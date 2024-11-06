# Automatic-Railway-Gate-Control-System
The automatic railway gate control system project was developed during my final year at Cooch Behar Government Engineering College in 2024. The primary goal of this project was to enhance railway safety by automating the operation of railway gates based on real-time train detection using Image Processing and Python. 
This project aims to create an automated railway gate control system using Raspberry Pi Zero 2W devices, camera modules, and a stepper motor, providing a reliable and cost-effective solution for railway crossings. The system includes two Raspberry Pi Zero 2W units equipped with cameras placed at a distance on both sides of a railway crossing, approximately 1 km away. These cameras detect approaching trains and update the train’s status and direction in real-time on a shared Google Sheet via the Google Sheets API. A third Raspberry Pi Zero 2W, connected to a stepper motor that controls the gate, continuously monitors the Google Sheet and adjusts the gate position based on the train’s location, either opening or closing it as needed. This Raspberry Pi acts as the server, while the cameras function as clients, establishing a client-server model. The project uses image processing for real-time train detection and automated gate control, aiming to improve safety and reduce the need for manual gate operation. This system is particularly suitable for rural areas with low train frequency, offering a scalable, efficient solution that enhances safety at rail crossings.
Component List Raspberry Pi Zero 2 W  , 5MP Raspberry Pi Zero 2W Camera Module , Type ‘A micro-USB’ 5v 2A adopter , 28BYJ-48 Stepper motor and ULN2003 stepper motor driver
The system operates by interfacing camera modules with Raspberry Pi Zero 2W units to detect trains approaching a railway crossing using real-time object detection with TensorFlow. Each camera-equipped Raspberry Pi (client node) monitors the track from a distance and detects an oncoming train, recording its direction (up/down) and sending this information to a Google Sheet via the Google Sheets API. A separate Raspberry Pi Zero 2W, connected to a 28BYJ-48 stepper motor (server node) through a ULN2003 motor driver, continuously reads the Google Sheet data. Based on the train's detected position and direction, the server activates the stepper motor to either open or close the gate automatically. This setup creates a fully automated gate control system using a client-server model, enhancing safety and eliminating the need for manual gate operation.
Client module for image processing (raspberry pi interfaced with 5mp camera module)
We implement tensor flow in our project.
The steps are given to install tensor flow model in raspberry pi
git clone https://github.com/EdjeElectronics/TensorFlow-Lite-Object-Detection-on-Android-and-Raspberry-Pi.git 

mv TensorFlow-Lite-Object-Detection-on-Android-and-Raspberry-Pi tflite1 

cd tflite1

sudo pip3 install virtualenv

python3 -m venv tflite1-env 

source tflite1-env/bin/activate

bash get_pi_requirements.sh

wget https://storage.googleapis.com/download.tensorflow.org/models/tflite/coco_ssd_mobilenet_v1_1.0_quant_2018_06_29.zip

unzip coco_ssd_mobilenet_v1_1.0_quant_2018_06_29.zip -d Sample_TFLite_model

python -m pip install --upgrade tflite-support==0.4.3

sudo apt-get install libopenblas-base

python3 TFLite_detection_webcam.py --modeldir=Sample_TFLite_model

Run image processing:
cd tflite1 && source tflite1-env/bin/activate && python3 TFLite_detection_webcam.py --modeldir=Sample_TFLite_model

 
4.1.2 Server node for controlling the gate (raspberry pi interfaced with 28BYJ-48 stepper motor)

Here is the code for controlling the stepper motor:
import RPi.GPIO as GPIO
import time
in1 = 17
in2 = 18
in3 = 27
in4 = 22
# careful lowering this, at some point you run into the mechanical limitation of how quick your motor can move
step_sleep = 0.002
step_count = 4096 # 5.625*(1/64) per step, 4096 steps is 360°
# direction = True # True for clockwise, False for counter-clockwise
# defining stepper motor sequence (found in documentation http://www.4tronix.co.uk/arduino/Stepper-Motors.php)
step_sequence = [[1,0,0,1],
                 [1,0,0,0],
                 [1,1,0,0],
                 [0,1,0,0],
                 [0,1,1,0],
                 [0,0,1,0],
                 [0,0,1,1],
                 [0,0,0,1]]
# setting up
GPIO.setmode( GPIO.BCM )
GPIO.setup( in1, GPIO.OUT )
GPIO.setup( in2, GPIO.OUT )
GPIO.setup( in3, GPIO.OUT )
GPIO.setup( in4, GPIO.OUT )
# initializing
GPIO.output( in1, GPIO.LOW )
GPIO.output( in2, GPIO.LOW )
GPIO.output( in3, GPIO.LOW )
GPIO.output( in4, GPIO.LOW )
motor_pins = [in1,in2,in3,in4]
motor_step_counter = 0 ;
def cleanup():
    GPIO.output( in1, GPIO.LOW )
    GPIO.output( in2, GPIO.LOW )
    GPIO.output( in3, GPIO.LOW )
    GPIO.output( in4, GPIO.LOW )
    GPIO.cleanup()
def stepper(direction, motor_step_counter):
    # the meat
    try:
        i = 0
        for i in range(step_count):
            for pin in range(0, len(motor_pins)):
                GPIO.output( motor_pins[pin], step_sequence[motor_step_counter][pin] )
            if direction=="ACL":
                motor_step_counter = (motor_step_counter - 1) % 8
            elif direction=="CL":
                motor_step_counter = (motor_step_counter + 1) % 8
            else: # defensive programming
                print( "uh oh... direction should *always* be either True or False" )
                cleanup()
                exit( 1 )
            time.sleep( step_sleep )
    except KeyboardInterrupt:
        cleanup()
        exit( 1 )
if __name__ == "__main__":
    while True:
        direction = input("Enter Direction 'CL' for Clockwise, 'ACL' for Anticlockwise: ")
        stepper(direction, motor_step_counter)
    cleanup()
    exit( 0 )
The project integrates software and hardware to create an automated railway gate control system. Camera modules connected to Raspberry Pi units act as client nodes, detecting approaching trains using TensorFlow Lite for real-time object detection. When a train is detected, the clients update a Google Sheet with the train's status and direction (up or down). A server Raspberry Pi connected to a stepper motor reads this data and automatically opens or closes the gate based on the train’s location. This client-server setup provides an efficient, hands-free solution for railway crossing safety, eliminating the need for human intervention.
The Automatic Railway Gate Control System project successfully demonstrates an efficient, low-cost method for automating railway gate operation. Using Raspberry Pi Zero 2W units, the system controls the gate via a stepper motor based on train detection data from image processing. While the processing speed is limited to 2 frames per second due to the Raspberry Pi’s low CPU power and lack of a dedicated GPU, the system effectively achieves the primary goal of automating gate operations for enhanced railway safety.
The Automatic Railway Gate Control System project demonstrates a successful, low-cost solution for automated gate control using Raspberry Pi Zero 2W units. Due to the device's limited CPU power and 512MB RAM, image processing speed is capped at 2 frames per second (fps). However, performance could significantly improve with upgraded hardware: a Raspberry Pi 5 could reach 20 fps, and adding a dedicated GPU like Google Coral could achieve up to 50 fps, enabling real-time processing. For demonstration, a 2 fps train video was used, and the system effectively controlled the stepper motor gate model based on train detection, validating the project's core functionality.
The Automatic Railway Gate Control System project addresses the need for safer and more efficient railway crossings, especially in rural or remote areas where manual control is often lacking. By automating gate operations, the system aims to prevent accidents and save government resources. However, limitations include reliance on internet connectivity, continuous power supply, and reduced accuracy in foggy conditions. Key advantages are automation, enhanced security, cost-effectiveness, and efficiency in gate operation. Future improvements could involve using more powerful hardware (e.g., Raspberry Pi 5 or GPUs), alternative power sources like solar energy, secure APIs, and additional sensors for greater accuracy and resilience.
