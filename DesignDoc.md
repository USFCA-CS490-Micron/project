# Senior Team Design Doc
### Andrew Moore, Jack Guo, Max Villarreal-Blozis, Nick Eastham
---
## Executive Summary
### The Project
This project is intended to provide a novel, privacy protecting, human interface to modern LLMs, utilizing a hybrid of local and cloud models.
### The Problem
Current AI wearables skimp on user interfaces and sacrifice privacy in the name of “simplicity;” they hand off all processing to cloud models, exposing user data, adding significant latency, and providing inferior performance across various use cases.

Many products treat models such as GPT-4o as general purpose and do not consider that various models are appropriate for only certain use-cases (for example, GPT-4o’s vision capabilities and processing are vastly inferior to Google Cloud Vision, while Google Gemini is inferior to GPT-4o in accurate and natural question-answering).

Using a hybrid local/cloud approach, we are able to provide quicker responses to simpler queries and save on bandwidth, while also being able to provide higher quality responses to more complex queries which require cloud models. 

### Related Work
This proposal is a conceptual amalgam of several privacy and security focused products, including Cloudflare’s [Tunnel and Warp](https://www.cloudflare.com/static/2aae70edac59c92cfddd7ba0a94c561d/Cloudflare_Whitepaper__Reference_Architecture_for_Internet-Native_Transformation.pdf), Apple’s hybrid [On-Device/Private Cloud Compute](https://security.apple.com/blog/private-cloud-compute/) and [iCloud Private Relay](https://www.apple.com/icloud/docs/iCloud_Private_Relay_Overview_Dec2021.pdf) (which is functionally a flavor of Cloudflare’s Tunnel and Warp) architectures.

This is essentially a rework of the architectures used by products like the Rabbit R1, Humane Ai Pin, and Brilliant Labs’s Noa; all of these products forego privacy and speed by making direct calls to cloud LLM providers’ APIs, as opposed to a private relay and local processing as proposed here.

---
## The Proposal
At the highest level, we propose a wearable AI system focused on privacy, speed, and quality of responses.
### Goals
- Create a wearable device
  - Hardware:
    - Ideally, Brilliant Frame glasses as they are a solid hardware foundation that will require little hardware engineering; using the Frame product will allow us to move more quickly on software development as we will not have to spend time on hardware development.
- Leverage on-device processing for simple queries.
  - Hardware:
    - Either a Raspberry Pi or other mobile device (user’s smartphone, for example)
  - Software:
    - Ollama for easy access to llama models and simple API
    - A method in which the RPi displays the GUI on the wearable and which it gets voice/vision data for processing.
- Build a private relay system to obfuscate the user’s activity when sending requests to a cloud provider (this is especially important for Google’s APIs).
  - Hardware:
    - Cloud services—low cost AWS/GCP/DigitalOcean VPS or docker/VM on a CS Lab system (such as medusa—we will have to work with the CS Systems team to determine if this is possible).
  - Software:
    - Basic web server for HTTP API relay
      - Flask may be a good option for this; packets will likely need to be programatically modified to replace origin IP and other identity-exposing data. The additional overhead imposed by Python will be negligible due to the simplicity of this system.
- Design and implement a user interface which reduces cognitive load via implicit design principles.
  - Hardware:
    - Not Applicable/Same as “Create a wearable device”
  - Software:
    - Dependent on hardware.
      - If Brilliant Frame is used, this will be a combination of Lua and Python.
      - If BasedHardware/omi is used, this will be significantly more complex, and will not allow for a GUI. For these reasons, we *strongly* suggest use of Brilliant Frame.
### Requirements
- A Raspberry Pi 5 with 8GB of RAM (needed for local processing) costs $80 (subtotal)
  - We are willing to self-fund the RPi given how significant it is to this project and that it would essentially absorb our entire budget.
- The private relay would require some kind of “server” hosting, at very least a docker container or two. We could potentially use the CS department’s existing infrastructure (maybe medusa) if resources are available, otherwise many cloud providers have student credit programs.
  - Because the private relay will not do any AI processing, it requires very little in resources to operate small-scale. 
- Our sponsor has provided a Brilliant Labs Frame, which cost $349/unit, though depending on assignments, we may only need the one. Worst case, our sponsor has said there’s a chance (of unknown probability) that we may be able to work with the company to get more.  
- Cloud models will likely require credits, though likely not many (primarily for OpenAI’s, which have no free tier). Our current architecture design will use OpenAI GPT-4o, Whisper, and tts-1, and Google Cloud Vision and Natural Language. We will (attempt to) use Whisper locally to improve response times.
  - OpenAI GPT (4o mini) will be used for conversational tasks and those which require advanced reasoning.
  - OpenAI Whisper will be used for voice recognition.
  - OpenAI TTS (tts-1) or a lower-quality local TTS will be used for text-to-speech (lower quality will increase greater cognitive load while the user processes the audio).
  - Google Cloud Vision will be used for tasks which require vision, such as “How many calories are in this?”, “What is this building?,” etc.
  - Google Natural Lanugage will be used to process queries which require advanced and current world understanding (in the “how many calories” example, GNL is able to provide accurate and timely information due to its access to Google’s massive information databases).
The primary contraint in this project will be budgetary due to hardware and API costs. 

### Non-Requirements
- Domain-specific requests (eg, “what song is this?”) would be nice to have, but are not required for functionality. These kinds of requests are best served by highly-specialized APIs, such as SoundHound/Shazam in the song-recognition example, and would require case-specific implementations.

---
## Stakeholders
Our primary focus will be the user and their experience. This project aims to create a consumer wearable platform that can aid the user in understanding the world around them

### Assignments (Tentative)

| Person | Primary Role                         |
|--------|--------------------------------------|
| Andrew | Interface and LLM-Tuning (Cognition) |
| Max    | Core Device (RPi)                    |
| Nick   | Private Relay (Internal Interface)   |
| Jack   | Private Relay (External Interface)   |

---
## User Stories

1. As a personal user, I will be able to make requests based on my worldly surroundings and receive timely and accurate answers based on context.
   - … I will be able to gather information and interact with the interface in an implicit manner, where it is aware of my needs and does not cause unnecessary cognitive load.
2. As an enthusiast, I will be able to modify and extend the platform based on my own wants and needs.
   - … I will be able to 
3. As someone who hates technology, I will not throw this across the room the first time I pick this up.

---
## Architecture
[stp-arch-proposal.pdf](Design-Doc/stp-arch-proposal.pdf)<!-- {"embed":"true", "preview":"true"} -->

- The HID (Glasses in this diagram) will collect user input (speech/audio/vision) and send this information to the RPi for processing.
- The RPi will:
  1. Use a fine-tuned distilBERT transformer to determine if a query can be handled on-device—for ex, “is this simple arithmetic (1+1)?”, or “does this require visual analysis?” (the former can be handled on-device, the latter requires cloud processing)
     - If the query requires cloud processing, the RPi will send the query and type of query (vision/complex/basic) to the private relay server.
  2. If this can be handled on device, either Llama or GPT-Neo will be used for local low-latency processing.
- Private Relay will determine the best model based on type sent by RPi and make requests to the external provider’s API.
  - If the query has a vision type, the relay will send the request to GCV, then, if up-to-date worldly information is necessary (eg “How many calories”), will request information from GNL.
  - If the query has an analytical/quesiton-answer type, the relay will use GPT-4o (eg “What is the stock price of AAPL and how does it compare to last year? Also, how is it performing compared to GOOG this month?”)
  - Domain-specific questions (eg, “What song is this?”) would require integration with various providers per domain (for ex, Shazam for song recognition). These integrations are not necessary, but nice-to-haves. We have put these in “##Non-Requirements” for that reason.

---
## Design Details

This project will leverage a natural human interface (for ex, glasses) connected to a Raspberry Pi, which will be the first step in handling a hybrid on-device/cloud processing. Once input is delivered to the RPi from the human interface device, the RPi will determine whether or not the query can be handled on device with Llama, or if it needs to be sent to an external provider. If the handler determines an external model is the best choice, a request will be sent to a private relay server, which will then communicate with the cloud provider. The private relay server will ensure the request cannot be tied back to the user by providing the bare minimum data to the external provider and obfuscating any and all personally identifying information in the request.

The vast majority of the work will lie in crafting a user interface which is simple and natural to use (most importantly requiring low cognitive load/processing), along with building software for the RPi to interact with the private relay. While the private relay will require significant work, the parts responsible for communication will be relatively simple due to the many packages available for this use. Depending on the interface device, work may also need to be done on creating a software interface between the interface device and RPi (the Frame SDK makes this extraordinarily simple, while Omi will make this slightly more complex as its APIs and SDKs are quite barebones).

---
## References

- Cloudflare Security and Privacy Architecture White Paper
  - https://www.cloudflare.com/static/2aae70edac59c92cfddd7ba0a94c561d/Cloudflare_Whitepaper__Reference_Architecture_for_Internet-Native_Transformation.pdf
- Apple Private Cloud Compute Architecture
  - https://security.apple.com/blog/private-cloud-compute/
- Apple iCloud Private Relay Architecture
  - https://www.apple.com/icloud/docs/iCloud_Private_Relay_Overview_Dec2021.pdf
- Brilliant Frame SDK
  - https://docs.brilliant.xyz/frame/building-apps-sdk/
- Brilliant Lua API
  - https://docs.brilliant.xyz/frame/building-apps-lua/
- Omi OpenGlass
  - https://github.com/BasedHardware/omi/tree/main/OpenGlass
- Google Cloud Vision API
  - https://cloud.google.com/vision/overview/docs/
- OpenAI Whisper API
  - https://platform.openai.com/docs/models/whisper
- OpenAI TTS API
  - https://platform.openai.com/docs/models/tts
- OpenAI GPT-4o/mini API
  - https://platform.openai.com/docs/models/gpt-4o
  - https://platform.openai.com/docs/models/gpt-4o-mini
- AWS Rekognition API
  - https://docs.aws.amazon.com/rekognition/latest/APIReference/Welcome.html
- Flask API
  - https://flask.palletsprojects.com/en/3.0.x/#api-reference
- distilBERT
  - https://huggingface.co/docs/transformers/en/model_doc/distilbert
- Hugging Face — Fine-tune a Pre-Trained Model
  - https://huggingface.co/docs/transformers/training
- Hugging Face — Text classification
  - https://huggingface.co/docs/transformers/tasks/sequence_classification
- (Maybe more?)


## Changelog
*Nothing to note yet*


## Meeting Log
- Met with Prof. Malensek, on Wednesday, 9/18, to discuss early progress.
- Met with Paul Lambert, project sponsor, on Friday, 9/20, to discuss basics of project.
