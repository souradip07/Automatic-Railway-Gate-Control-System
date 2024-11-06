# Automatic-Railway-Gate-Control-System
The Automatic Railway Gate Control System, developed during the final year at Cooch Behar Government Engineering College in 2024, aims to enhance railway safety by automating gate operations using real-time train detection with image processing and Python. Utilizing Raspberry Pi Zero 2W devices, camera modules, and a stepper motor, the system provides an efficient and cost-effective solution for rural railway crossings.

The setup includes two Raspberry Pi Zero 2W units equipped with cameras positioned 1 km away from the railway crossing on both sides to detect approaching trains. When a train is detected, its status and direction (up/down) are updated in real-time on a shared Google Sheet via the Google Sheets API. A third Raspberry Pi, connected to a stepper motor that controls the gate, acts as the server. This unit continuously monitors the Google Sheet to adjust the gate position based on the train’s location, either opening or closing it as needed. This client-server model enhances safety by automating the gate system, making it ideal for rural areas with low train frequency.

The system's hardware includes Raspberry Pi Zero 2W units, 5MP camera modules, a 28BYJ-48 stepper motor, and a ULN2003 motor driver. TensorFlow is used for real-time object detection, enabling each client node to monitor the track, update the train's status, and trigger the gate’s operation through the server, creating a fully automated, efficient gate control system.
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
The Automatic Railway Gate Control System project automates the operation of railway gates using Raspberry Pi Zero 2W units, camera modules, and a stepper motor. The camera modules, functioning as client nodes, use TensorFlow Lite for real-time object detection to monitor trains. When a train is detected, the clients update a Google Sheet with the train’s status and direction, which is then read by the server Raspberry Pi to control the gate’s position automatically. This system eliminates the need for human intervention, enhancing safety at railway crossings.

The project successfully demonstrates a low-cost, efficient solution for automated gate control. Despite the limited processing speed of 2 frames per second due to the Raspberry Pi's hardware limitations, the system effectively automates gate operations. With upgraded hardware such as a Raspberry Pi 5 or a dedicated GPU like Google Coral, the system’s image processing speed could be improved significantly, achieving real-time performance.

This automated system is especially beneficial for rural or remote areas where manual control is insufficient, preventing accidents and reducing costs. Limitations include dependency on internet connectivity, a continuous power supply, and reduced performance in foggy conditions. The system's advantages include automation, security, cost-effectiveness, and operational efficiency. Future improvements may include more powerful hardware, alternative energy sources like solar power, enhanced security measures, and additional sensors to improve accuracy and resilience.
