# Raspberry Pi Color Cycler with Sense Hat

_This repository is part of the Twitch series, "Voice + Robotics", found at [https://twitch.tv/amazonalexa/](https://twitch.tv/amazonalexa)._

This is device (Rasperry Pi) code and Alexa Skills Kit files for creating a skill to interact with a Raspberry Pi. This is heavily based off of the [Alexa Gadgets Raspberry Pi Color Cycler](https://github.com/alexa/Alexa-Gadgets-Raspberry-Pi-Samples/tree/master/src/examples/colorcycler/) sample, but updated to work with a [Sense Hat](https://www.raspberrypi.org/products/sense-hat/) rather than GPIO bulbs and buttons. The Sense Hat provides an 8x8 LED matrix and sensors like an accelerometer, gyroscope, and more. The LED matrix will be used to provide feedback from communication between Raspberry Pi and Echo.

You will need:

* A Raspberry Pi (version 3 or above)
* [Sense Hat](https://www.raspberrypi.org/documentation/hardware/sense-hat/)
* An Amazon Web Services (AWS) account
* An Alexa Developer account

## Steps

### Register your gadget

Follow the instructions in [Register a Gadget](https://github.com/alexa/Alexa-Gadgets-Raspberry-Pi-Samples/blob/master/README.md#registering-a-gadget-in-the-alexa-voice-service-developer-console) to register your gadget with the Alexa Voice Service (AVS) using the developer portal.

- Take note of your Amazon ID and Alexa Gadget Secret when you've completed the initial setup

### Setup Your Raspberry Pi

You'll first want to setup your Raspberry Pi with the [latest version of Raspbian](https://www.raspberrypi.org/downloads/raspbian/). 

Then, you'll need to install the Alexa Gadgets Toolkit. The easiest way to setup the Alexa Gadgets Toolkit is to clone the sample repo at [https://github.com/alexa/Alexa-Gadgets-Raspberry-Pi-Samples](https://github.com/alexa/Alexa-Gadgets-Raspberry-Pi-Samples) and follow the instructions to configure your Raspberry Pi and register it as a gadget. 

Finally, clone this repository to your Raspberry Pi. 

### Create a Custom Skill with the Alexa Skills Kit

1. [Create a Skill](https://developer.amazon.com/alexa/console/ask/create-new-skill) using the Alexa Skills Kit, choosing "Custom" as the model. 

2. Scroll down to "Choose a method to host your skill's backend resources", and choose "Provision Your Own." Then scroll back to the top then click the **Create Skill** button.

3. For "Choose a Template" choose "Start from Scratch," then click the **Choose** button.

4. In the left menu, choose "JSON Editor," then replace the contents with the file found in [models/en-US.json], then click the **Save Model** button.

5. Now click "Interfaces" in the left menu, and scroll down to "Custom Interface Controller" and enable it by clicking the button at the end of the row. Click the **Save Interfaces** button.

6. Click "Endpoint" in the left menu, and select the "AWS Lambda ARN" radio button. Copy to the clipboard the value next to "Your Skill ID". It will look similar to:

        amzn1.ask.skill.f873f5c1-9000-d74d-0000-5853abcdefg

We'll temporarily leave this page now and create a Lambda function. Keep it open and follow the next section.

### Create an AWS Lambda Function

AWS Lambda will be the endpoint of the skill you just created. It will interact with an Echo device to pass directives to and from your Echo to the Raspberry Pi. 

1. Leave the Alexa Skills Kit tab/window open, and open a new tab/window to [the AWS Lambda Console](https://console.aws.amazon.com/lambda/home?region=us-east-1#/create). (This opens a console to the `us-east-1` region. You may change this to the region of your choice by using the region dropdown in the top right corner of the console).

2. Choose "Author from scratch" and give your function a name, e.g., "ColorCycler". Choose a runtime of "Python 3.6". For "Execution Role," leave it at "Create a new role with basic Lambda permissions," then click the **Create Function** button.

3. You'll be taken to your Lambda function's main console. Scroll down to the _Function Code_ section, and under "Code entry type" choose "Upload a .zip file" from the dropdown. Click the **Upload** button and choose the file `function.zip` found at [/skill/lambda/function.zip]. Click the **Save** button at the top of the window to upload the file.

We uploaded all of our required dependencies by bundling them in a [deployment package](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html). We'll now replace the simple contents of the code editor with our full code.

4. Copy the Python code from [/lambda/lambda_function.py](/lambda/lambda_function.py) and replace the code in the "lambda_function.py" tab in the editor. Your code should now start with the following:

        #
        # Copyright 2019 Amazon.com, Inc. or its affiliates.  All Rights Reserved.
        # These materials are licensed under the Amazon Software License in connection with the Alexa Gadgets Program.
        # The Agreement is available at https://aws.amazon.com/asl/.
        # See the Agreement for the specific terms and conditions of the Agreement.
        # Capitalized terms not defined in this file have the meanings given to them in the Agreement.
        #
        import logging.handlers
        import requests
        import uuid
        ...

5. Click the **Save** button at the top of the window.

### Connect Lambda with the Alexa Skills Kit

1. At the top of the Lambda console in the _Designer_ section, click the **+ Add Trigger** button. 

2. In the dropdown under "Trigger Configuration," choose "Alexa Skills Kit". In the "Skill ID" box, paste the skill ID you copied from the Alexa Skills console, then click the *Add* button.

2. At the top-right of your Lambda console, you'll find an _ARN_ (Amazon Resource Name). It will look similar to:

        arn:aws:lambda:us-east-1:1234567890:function:ColorCycler

Copy this value.

3. Leave this tab/window open and return to the tab/window for the Alexa Skills Kit. If it's not already selected, click "Endpoint" in the left menu, and select the "AWS Lambda ARN" radio button. In the box next to "Default Region," paste the ARN you just copied. Click the **Save Endpoints** button at the top of the window.

4. From the left menu, select "Invocation," then click the **Build Model** button.

### Test Your Function

1. Back in your Alexa Skills Kit developer console tab/window, click the "Test" menu item near the top of the page, then click the dropdown to change "Off" to "Development".

2. On the left of the window in the _Alexa Simulator_, type `open color cycler` in the box and press Enter. Alexa should respond indicating that no gadgets were found (since this is a simulator). You can also see the JSON for Alexa's response, e.g.,

        {
            "body": {
                "version": "1.0",
                "response": {
                    "outputSpeech": {
                        "type": "SSML",
                        "ssml": "<speak>No gadgets found. Please try again after connecting your gadget.</speak>"
                    },
                ...

3. Now, open a terminal on your Raspberry Pi so that you can run the device code.

    cd RaspberryPiColorCycler/device
    mv colorcycler-sample.ini colorcycler.ini

4. Edit the contents of colorcycler.ini so and replace the default values with your `amazonId` and `alexaGadgetSecret`.

5. Run the colorcycler.py file. If you're runnign it for the first time, follow the onscreen instructions to pair your device with your Echo. When successful, you should see a `connected` message, e.g.,

```bash
pi@raspberrypi:~/RaspberryPiColorCycler/device $ python3 colorcycler.py
INFO:agt.alexa_gadget:Attempting to reconnect to Echo device with address: 74:C2:46:CE:22:11
INFO:agt.alexa_gadget:Connected to Echo device with address: 74:C2:46:CE:22:11
```

6. Once connected, invoke the skill by telling your Echo, "Alexa, open color cycler". Listen to the instructions, and click the center of the joystick on the Sense Hat when you want to report the color you've selected back to Alexa.


## Resources

- [https://github.com/alexa/Alexa-Gadgets-Raspberry-Pi-Samples](https://github.com/alexa/Alexa-Gadgets-Raspberry-Pi-Samples)