<img src="http://developer.download.nvidia.com/notebooks/dlsw-notebooks/riva_sample_virtual-assistant-dialogflow-readme/nvidia_logo.png" style="width: 90px; float: right;">

# Virtual Assistant (with Google Dialogflow)

This Virtual Assistant (with Google Dialogflow) sample application demonstrates the integration of Google Dialogflow and Riva Speech Services in the form of a weather chatbot web application.

In this sample, we use Riva for ASR and TTS and Google Dialogflow for NLP and Dialog Management (DM).

## Demo Video

To see how the Riva and Google Dialogflow weather chatbot service works, the demo video can be found [here](https://youtu.be/9pG8WDLjWMk).

## Implementation

At a high-level, the integration takes advantage of the native API support of Google Dialogflow and gRPC support in
Riva. The Weatherbot Client coordinates the workflow with Riva Services and Dialogflow, then interacts with the
end-user via a web UI. There are three primary parts to this solution; Riva AI Services, Dialogflow Weatherbot, and the Weatherbot Client
application.

Here is the implementation at a high-level:

- Riva AI Services
  - Exposes Speech Services (ASR/NLP/TTS) over gRPC endpoints.
  - Needs a GPU.

- Riva and Dialogflow Chatbot

  - Dialogflow Weatherbot
    - Exposes API endpoints to communicate with the chatbot.
    - Takes user text as input and returns a response.
    - Responsible for fulfillment, when needed.
    - Runs on GCP.

  - Weatherbot Client application
    - Includes the Riva Client Python library.
    - Communicates with Riva AI Services and Dialogflow Weatherbot over gRPC and REST API endpoints respectively.
    - Pipelines ASR, NLP, TTS, and dialog manager functionalities.
    - Contains the Weatherbot Client application (web UI and web service).
    - Does not need GPUs.

### Architecture

![Riva and Dialogflow virtual assistant architecture](img/riva-dialogflow.png "Riva and Dialogflow virtual assistant architecture")

The above diagram shows the architecture of the Riva and Dialogflow Weatherbot. Audio input from the user is
collected through the microphone by the web UI of the Weatherbot Client application. The input audio from the user
is sent to Riva AI Services for ASR, by the client application. Riva AI Services returns the transcribed text
back to the Client application. The transcribed text from Riva AI Services is then sent to the Dialogflow Weatherbot
(running on GCP). The Dialogflow Weatherbot returns the appropriate response for the text. The response text is then
sent to Riva AI Services for TTS. A voice response is returned back to the client application, which is then played
on the user’s speakers by the web UI.

### Code Structure

This section shows the high-level code structure of the Weatherbot Client application (in
Riva and Dialogflow Chatbot).

- `asr.py`
  - This file contains the functionality to make the gRPC call to Riva ASR, using the Riva Python Client
    libraries with the audio snippet and returns the text transcript.
  - ASR is used in streaming mode

- `dialogflow.py`
  - This file contains the functionality to make an API call to Dialogflow, with the user input and sender ID
    and returns a text response obtained by Dialogflow.

- `tts.py` and `tts_stream.py`
  - These files contain the functionality to make the gRPC call to Riva TTS, using the Riva Python Client
    libraries, with a text snippet, and returns the corresponding audio speech.
  - TTS can be used in either Batch or Streaming mode, depending on whether `tts.py` or `tts_stream.py` is used.
    This can be set by changing the import statements in lines 12 and 13 in the `virtual-assistant-dialogflow/dialogflow-riva-weatherbot-webapp/riva/chatbot/chatbot.py` script.

- `chatbot.py`
  - This file contains the Chatbot class which is responsible for pipelining all the ASR, TTS and Dialogflow
    operations.
  - Creates one instance of the Chatbot class per conversation.

## Requirements and Setup

### Requirements:
Before you try running the Riva client, ensure you meet the following requirements:
1. You have access and are logged into NVIDIA NGC. For step-by-step instructions, refer to the [NGC Getting Started Guide](https://docs.nvidia.com/ngc/ngc-overview/index.html#registering-activating-ngc-account).
2. Python 3.8 (Support for other Python versions will be added in a future release).

### Setup:

1. Clone [Riva Sample Apps repository](https://github.com/nvidia-riva/sample-apps):
```
	git clone https://github.com/nvidia-riva/sample-apps.git
```

2. Enter Riva and Dialogflow Virtual Assistant directory:
```
	cd sample-apps/virtual-assistant-dialogflow
```

3. Create parent directory for the Python virtual environment we will be using for this sample:
```
	mkdir pythonenvs
```

4. Create and activate Python virtual environment for this sample. We will be using this virtual environment for all dependent libraries, including Riva client library, weatherbot web application libraries, and Google Dialogflow client library:
```
	python3 -m venv pythonenvs/dialogflow
	. pythonenvs/dialogflow/bin/activate
```

5. Install the libraries necessary for the virtual assistant, including the Riva client library:
	1. Upgrade `pip`:
	```
		pip3 install -U pip
	```
	2. Install Riva client libraries:		
	```
		pip3 install nvidia-riva-client
	```
	3. Install weatherbot web application dependencies. `requirements.txt` captures all Python dependencies needed for weatherbot web application:
	```
		pip3 install -r requirements.txt # For Python 3.8
	```

6. [Set up](https://cloud.google.com/dialogflow/es/docs/quick/setup) Google Dialogflow. The entire set up process for Dialogflow consists of multiple steps and can take some time to complete. <br> At the end of this step, you would have setup a Google Project, installed and initialized gcloud CLI, and installed the Google Dialogflow client library in the `dialogflow` virtual environment that we created in the previous step.

7. In `virtual-assistant-dialogflow`'s `config.py`, update `PROJECT_ID` parameter with your project ID. To find your Project ID, perform the following steps:
	1. In the [Google Cloud Platform (GCP) Dashboard](https://console.cloud.google.com/home/dashboard), select your project from the top-left drop-down, found on the right side of the GCP banner.
	2. Under the **DASHBOARD** tab, the Project ID can be found in the *Project Info* section.

8. Deactivate Weatherbot web application's `dialogflow` Python virtual environment:
```
	deactivate
```


## Running the Demo

1. Download the Riva Quick Start scripts, if not already done. `x.y.z` is the Riva Speech Skills version number - The latest Riva version number can be found in the [Riva Quick Start Guide](https://docs.nvidia.com/deeplearning/riva/user-guide/docs/quick-start-guide.html#)'s [Local Deploymnent using Quick Start Scripts section](https://docs.nvidia.com/deeplearning/riva/user-guide/docs/quick-start-guide.html#local-deployment-using-quick-start-scripts)
```
	ngc registry resource download-version "nvidia/riva/riva_quickstart:x.y.z"
```

2. Start the Riva Speech Server, if not already done. Follow the steps in the [Riva Quick Start Guide](https://docs.nvidia.com/deeplearning/riva/user-guide/docs/quick-start-guide.html).

3. Navigate to the Riva and Dialogflow Virtual Assistant directory in the Riva sample-apps github repository that you cloned in the [Setup section](README.md#setup)'s, step 1.
```
	cd sample-apps/virtual-assistant-dialogflow
```

4. Modify the API endpoint setting in the code base for inter-service communication - Update the `config.py` script for inter-service communication.
```
	riva_config = {
	    ...
	    "RIVA_SPEECH_API_URL": "[riva speech service host IP]:50051",
	    ...
	}
```
For example:
```
	# uncomment and populate the section below
	riva_config = {
		...
		"RIVA_SPEECH_API_URL": "10.20.30.40:50051",
		...
	}
```

5. Initialize and start the Dialogflow Weatherbot.
	1. Follow the steps [here](https://cloud.google.com/dialogflow/es/docs/quick/build-agent#create-an-agent) to create an agent.
	2. Click the **Setting** button next to the agent name in the Dialogflow console. Under the **Export and Import** tab, choose **Restore From ZIP** and upload the zipped folder from your host at `sample-apps/virtual-assistant-dialogflow/dialogflow-weatherbot/dialogflow-weatherbot.zip`.
	3. Add fullfillment.
		1. Open the **Fulfillment** section and enable the **Inline Editor** in the Dialogflow console.
		2. Copy and paste the contents of the `sample-apps/virtual-assistant-dialogflow/dialogflow-weatherbot/fulfillment/index.js` into `index.js` under the **Inline Editor**.
		3. Copy and paste the contents of the `sample-apps/virtual-assistant-dialogflow/dialogflow-weatherbot/fulfillment/package.json` into `package.json` under the **Inline Editor**.
		4. In `index.js`, at line 4, update the `weatherstack_APIkey` with your Weatherstack API key. A new Weatherstack API key can be obtained from [here](https://weatherstack.com/).

6. Start Riva and Dialogflow virtual assistant web application.
	1. If not already done, update `config.py` with the Dialogflow weatherbot's Project ID as described in the [Setup section](README.md#setup), step 7.
	2. Activate the `dialogflow` Python environment.
	```
		. /pythonenvs/dialogflow/bin/activate
	```
	3. Navigate to the chatbot web application folder.
	```
		cd dialogflow-riva-weatherbot-webapp
	```
	4. Start the chatbot web server.
	```
		python3 main.py
	```
	5. Open the interface in a web browser `https://<riva chatbot server host IP>:6006/rivaWeather`, e.g. [https://0.0.0.0:6006/rivaWeather](https://0.0.0.0:6006/rivaWeather).

## Sample Conversations

### Sample 1

**Virtual Assistant:** Hi, welcome to RIVA weather service. How can I help you?

**User:** How's the weather?

**Virtual Assistant:** For which location?

**User:** In Seattle.

**Virtual Assistant:** It is Partly cloudy in Seattle at the moment. The temperature is 16 degrees, the humidity is
97 percent and the wind speed is 0 miles per hour.

### Sample 2

**Virtual Assistant:** Hi, welcome to RIVA weather service. How can I help you?

**User:** How's the weather in Paris?

**Virtual Assistant:** It is Clear in Paris at the moment. The temperature is 20 degrees, the humidity is 73 percent
and the wind speed is 9 miles per hour.

### Sample 3

**Virtual Assistant:** Hi, welcome to RIVA weather service. How can I help you?

**User:** Hi

**Virtual Assistant:** Hey! How can I help you?

**User:** What is your name?

**Virtual Assistant:** My friends call me Ja-Di.

### Sample 4

**Virtual Assistant:** Hi, welcome to RIVA weather service. How can I help you?

**User:** Good morning

**Virtual Assistant:** Hey! How can I help you?

**User:** See you around

**Virtual Assistant:** Bye

## Limitations

- The provided samples are not complete virtual assistants; but are intended as simple examples of how to build
  basic task-oriented chatbots with Riva. Consequently, the intent classifier and slot filling models have been
  trained with small amounts of data and are not expected to be highly accurate.
- The Dialogflow Weatherbot sample supports intents for cloudy, humidity, rainfall, snow, sunny, temperature,
  weather and windy checks. It does not support general conversational queries or other domains.
- The Dialogflow Weatherbot supports only 1 slot for `city`. It doesn’t take into account the day associated
  with the query while providing the response.
- This sample supports up to four concurrent users. This restriction is not because of Riva, but because of the
  web framework (Flask and Flask-ScoketIO) that is being used by the client web application. The socket connection
  to stream audio to (TTS) and from (ASR) the user is unable to sustain more than four concurrent socket connections.

## License

The [NVIDIA Riva License Agreement](https://developer.nvidia.com/riva/ga/license) is included with the product. Licenses are also available along with the model application zip file. By pulling and using the Riva SDK container, downloading models, or using the sample applications here, you accept the terms and conditions of these licenses.   <br>
Here is the license information for the libraries we are using in this sample:
1. [google-cloud-dialogflow](https://github.com/googleapis/python-dialogflow) - The License for this library can be found [here](https://github.com/googleapis/python-dialogflow/blob/master/LICENSE).
2. [actions-on-google](https://github.com/actions-on-google/actions-on-google-nodejs) -  The License for this library can be found [here](https://github.com/actions-on-google/actions-on-google-nodejs/blob/HEAD/LICENSE).
3. [firebase-admin](https://www.npmjs.com/package/firebase-admin) -  The License for this library can be found [here](https://www.npmjs.com/package/firebase-admin#license).
4. [firebase-functions](https://github.com/firebase/firebase-functions) -  The License for this library can be found [here](https://github.com/firebase/firebase-functions/blob/HEAD/LICENSE).
5. [dialogflow](https://github.com/googleapis/nodejs-dialogflow) -  The License for this library can be found [here](https://github.com/googleapis/nodejs-dialogflow/blob/master/LICENSE).
6. [dialogflow-fulfillment](https://github.com/dialogflow/dialogflow-fulfillment-nodejs) -  The License for this library can be found [here](https://github.com/dialogflow/dialogflow-fulfillment-nodejs/blob/master/LICENSE).
